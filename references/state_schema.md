# State Schema Reference

## .state file structure

The `.state` file is the single source of truth for project phase and loop state. It MUST be fsynced after every write.

```json
{
  "project_slug": "string",
  "current_phase": "IDLE|IDEATION|BUILD|STRESS|SECURITY|DEPLOY|CORRECTION|ESCALATED",
  "checkpoint_phase": "IDEATION|BUILD|STRESS|SECURITY|DEPLOY",
  "correction_checkpointed": "IDEATION|BUILD|STRESS|SECURITY|null",
  "loop_count": { "ideation": 0, "build": 0, "stress": 0, "security": 0 },
  "last_checkpoint": "ISO8601",
  "last_fsync_verified": "ISO8601",
  "errors": [],
  "artifacts": {
    "plan": ".team/{slug}/plan.md",
    "build": [".team/{slug}/build/frontend/", "..."],
    "stress_report": ".team/{slug}/stress-report.md",
    "security_clearance": ".team/{slug}/security-clearance.md"
  },
  "artifact_manifests": [
    {
      "artifact_id": "uuid",
      "source_plan_version": "hash",
      "git_commit": "hash or 'unversioned'",
      "branch": "name",
      "produced_at": "ISO8601",
      "builder_agent": "Frontend|Backend|DevOps|QA",
      "artifact_path": "relative path",
      "manifest_hash": "sha256 of manifest itself for tampering detection",
      "compliance_attestation": {
        "plan_checklist": [{ "item": "", "status": "implemented|partial|missing", "evidence": "" }],
        "all_passed": true
      },
      "errors": [],
      "warnings": []
    }
  ]
}
```

**Key rules:**
- `loop_count` is the single source of truth for correction tracking
- State transitions use `loop_count < max_loops` to determine if CORRECTION is allowed
- `last_fsync_verified` is set ONLY after confirmed fsync
- `errors` is accumulated across the session, cleared on new phase entry

---

## Relay Lock Implementation

Concurrency control using atomic file-based lock:

1. On /team invocation: attempt to acquire `.team/{project}/.lock` with atomic SET NX EX=3600
2. If lock EXISTS: reject new /team with error "Team session already active for {project}. Use /team stop first."
3. If lock ACQUIRED: proceed with relay lock
4. Lock is released on: team conclusion, /team stop, orchestrator error, or timeout (3600s)
5. Lock file contains: `{owner_pid, session_id, started_at, current_phase, expires_at}`
6. On orchestrator error: attempt graceful lock release; if write fails, lock expires at TTL

**TTL Extension:**
- On each successful .state checkpoint write, extend the lock TTL by 3600s
- BEFORE spawning agents for any phase, verify lock TTL > estimated phase duration + 300s buffer; if not, extend TTL first
- **For distributed locks (Redis/etcd):** issue EXPIRE with new TTL=3600s, or re-acquire with same session_id
- **For file-based locks:** delete and recreate lock file with new 3600s expiry (atomic mkdir or equivalent)
- **The mtime of a stale lock file does NOT extend a semaphore TTL** — you must explicitly refresh the lock's expiry

7. On /team stop: release lock immediately and explicitly

---

## fsync Write Policy

Every .state write must follow this sequence:

1. Write `.state.tmp` (temp file in same directory)
2. `fsync(.state.tmp)` — verify write succeeded to disk
3. IF fsync fails: delete `.state.tmp`, ROLL BACK in-memory state to previous checkpoint, write error to `.state.errors[]`, exit with `StateWriteError`
4. IF fsync succeeds: rename `.state.tmp` → `.state` (atomic rename)
5. Verify fsync succeeded by checking file mtime matches current time (within 1s tolerance) — if not, retry once
6. IF retry succeeds: set `last_fsync_verified` to current ISO8601 and proceed. IF retry fails: delete `.state.tmp`, ROLL BACK in-memory state to previous checkpoint, write error to `.state.errors[]`, exit with `StateWriteError`
7. ONLY advance in-memory state AFTER confirmed fsync

**Atomic batch (state + registry):**
- Write `.state.tmp` and `index.md.tmp` simultaneously
- fsync both files
- Only after both succeed: rename both atomically
- If either fsync fails: rollback BOTH, exit with `StateWriteError`

---

## session-memory.json Schema

Written by Orchestrator incrementally during EVALUATE step, finalized by MemoryCurator at session end. Lives at `.team/{project-slug}/session-memory.json`.

```json
{
  "project_slug": "string",
  "session_started_at": "ISO8601",
  "session_ended_at": "ISO8601|null",
  "mode": "full|plan|lite|debug|pipeline|full-ultrathink|plan-ultrathink|debug-ultrathink|lite-ultrathink",
  "ultrathink_aggregated": {
    "chain_attack_effectiveness_count": { "exposed_critical_flaws": 0, "superficial": 0, "no_flaws_found": 0 },
    "steelman_quality_count": { "strong": 0, "weak": 0, "none": 0 },
    "agents_with_critical_flaws": ["string"],
    "genuine_position_updates": 0
  },
  "correction_summary": {
    "corrections_count": 0,
    "root_causes": ["string"],
    "fix_strategies": ["string"]
  },
  "topic_tags": ["string"],
  "key_outcomes": ["string"],
  "loop_counts": { "ideation": 0, "build": 0, "stress": 0, "security": 0 }
}
```

---

## correction-log.md Schema

Written on CORRECTION entry, updated on CORRECTION resolution. Lives at `.team/{project-slug}/correction-log.md`.

```markdown
# Correction Log — {project_slug}

## correction/{phase}/{loop_number}/{timestamp}
- phase: "IDEATION|BUILD|STRESS|SECURITY"
- loop_number: 1|2
- root_cause: "string"
- failure_chain: ["symptom", "intermediate cause", "root cause"]
- fix_strategy: "string"
- resolution: "succeeded|escalated|abandoned"
- related_judgments: ["judgment_id"]
- ultrathink_reasoning_flaws: ["string"]   # from ChainAttacker if ultrathink
- created_at: "ISO8601"
- resolved_at: "ISO8601|null"
```

---

## learnings.md Schema

Cross-session append-only learnings database at `.team/learnings.md`.

```markdown
## learning/{uuid}
- project: {slug}
- category: {primary tag}        # auth, websocket, performance, security, frontend, backend
- topic_tags: [{tag1}, {tag2}]
- rule: "{prescriptive rule}"
- mistake: "{what went wrong}"
- correction: "{how it was fixed}"
- source: "correction|escalation|post_mortem|insight"
- source_phase: "IDEATION|BUILD|STRESS|SECURITY|DEPLOY"
- times_applied: 0
- created_at: "ISO8601"
- last_applied_at: "ISO8601|null"
```

**Source values:**
- `correction` — extracted from a resolved CORRECTION entry
- `escalation` — extracted from ESCALATED resolution
- `post_mortem` — MemoryCurator inference from agent outputs
- `insight` — ultrathink reasoning pattern that survived full scrutiny

---

## pattern-library.md Schema

Auto-generated by `/team rebuild-patterns`. Lives at `.team/pattern-library.md`.

```markdown
# Pattern Library
# Rebuilt on: {ISO8601}

## category/{category_name}
- pattern_count: {N}
- common_mistakes: ["{mistake1}", "{mistake2}"]
- prevention_strategy: "{aggregated guidance}"
- learnings: ["{learning_id1}", "{learning_id2}"]
```