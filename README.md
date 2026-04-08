# /team — Multi-Agent Orchestration Skill for Claude Code

A structured multi-agent debate and build pipeline framework. Coordinates Analyst, Designer, Implementer, Contrarian, and specialist agents through IDEATION → BUILD → STRESS → SECURITY → DEPLOY with human checkpoints, adversarial testing, and persistent cross-session memory.

---

## Installation

The skill is a single YAML file plus a references document. Copy into your Claude Code configuration:

```
~/.claude/skills/team/
├── SKILL.md                      # Main skill definition (orchestrator prompt + agents + schemas)
└── references/
    └── state_schema.md          # State file schemas, relay lock, fsync policy
```

After copying, activate with `/team` in any Claude Code session.

---

## Quick Start

```bash
# Quick binary decision — 1 loop, 2 agents
/team quickly should we add Redis caching to our API

# Full architectural debate — 2 loops, 4 agents
/team should we migrate from REST to GraphQL

# Plan a feature build
/team plan add real-time notifications

# Full build pipeline (dry-run first)
/team all add real-time notifications --dry-run
/team all add real-time notifications --confirm
```

---

## Core Concepts

### Relay Lock

Once `/team` is invoked, the main Claude Code agent becomes a **relay** — it forwards all messages to the Orchestrator sub-agent and cannot independently spawn agents, write files, or run commands. This prevents contamination of debate or execution by the parent session.

The lock releases on: team conclusion, `/team stop`, orchestrator error, or timeout (3600s).

### Debate Modes

| Mode | Agents | Loops | Best For |
|------|--------|-------|----------|
| **full** | Analyst, Designer, Implementer, Contrarian | 2 | Complex architectural decisions |
| **lite** | Advocate, Skeptic | 1 | Quick binary decisions |
| **debug** | Investigator, RootCauseAnalyst, FixStrategist, Contrarian | 2 | Root cause investigation |
| **plan** | Analyst, Designer, Implementer, Contrarian | 2 | Planning and build recommendations |
| **ultrathink** | All above, but ChainAttacker → Steelman → SelfCritique | 2 | High-stakes: auth, payments, security, compliance |

**Ultrathink** auto-activates when the topic contains keywords: `security`, `auth`, `payment`, `OAuth`, `JWT`, `GDPR`, `infrastructure`, `compliance`, etc. It runs 5 phases instead of 4: Reasoning Chain → Chain Attack → Steelman → Self-Critique + Revision → Synthesis.

### Pipeline Phases

```
IDLE → IDEATION → BUILD → STRESS → SECURITY → DEPLOY
              ↑         ↓
           CORRECTION ←←←←←←←←←←←
```

- **IDLE → IDEATION**: `/team [x]` or `/team plan [x]`
- **IDEATION → BUILD**: requires explicit user confirmation (HARD-GATE)
- **BUILD**: 4 parallel builders (Frontend, Backend, DevOps, QA)
- **STRESS**: LoadTester + EdgeCaseAnalyst
- **SECURITY**: SecurityAuditor + DependencyScanner
- **DEPLOY**: Orchestrator approval gate — both STRESS and SECURITY must clear

Each phase allows 2 loops before escalation. HARD-GATE between IDEATION and BUILD is non-negotiable — the Orchestrator must prompt for confirmation and cannot auto-advance.

### Memory System

Persistent cross-session memory, inspired by pro-workflow:

| File | Purpose |
|------|---------|
| `.team/learnings.md` | Cross-session append-only learnings database |
| `.team/pattern-library.md` | Aggregated patterns by category (rebuilt via `/team rebuild-patterns`) |
| `.team/{slug}/session-memory.json` | Per-session capture: ultrathink stats, correction summary, topic tags |
| `.team/{slug}/correction-log.md` | Enhanced CORRECTION entries with root cause + fix strategy |

**MemoryCurator** runs automatically at session end (before relay lock release). It:
- Extracts `[LEARN]` blocks from agent outputs
- Deduplicates against existing learnings
- Updates `times_applied` on replayed learnings
- Writes finalized `session-memory.json`

**Pre-flight query** runs automatically at every IDEATION start — surfaces top 5 relevant past learnings into the Orchestrator context, with escalation warnings for previously-failed categories.

---

## Commands

### Debate Commands

```
/team [x]                    Full debate — 2 loops, 4 agents (Position → Contrarian → Rebuttal → Synthesis)
/team quickly [x]            Lite debate — 1 loop, 2 agents (Advocate + Skeptic)
/team findbug [x]            Debug — Investigator + RootCauseAnalyst + FixStrategist + Contrarian
/team plan [x]               Plan — Analyst + Designer + Implementer + Contrarian → FinalPlan
```

### Pipeline Commands

```
/team all [x]                Full pipeline with checkpoints (requires --confirm)
/team build [x]              BUILD phase only (requires plan.md)
/team stress [x]             STRESS phase only (requires build artifacts)
/team security [x]           SECURITY phase only (requires stress report)
/team status [proj]           Show current phase, artifacts, loop counts
/team resume [proj]           Resume from last checkpoint
/team stop                    Abort active session, release relay lock
```

### Memory Commands

```
/team replay [topic]          Surface relevant past learnings for a topic (read-only)
/team rebuild-patterns         Regenerate pattern-library.md from learnings.md (read-only)
```

### Flags

```
--dry-run                     Show full phase tree without executing
--confirm                    Execute pipeline after --dry-run confirmation
--from <path>                Use existing artifacts at path (skip upstream phases)
--force                       Skip prerequisite checks
--skip <phase>               Skip a specific phase
```

---

## State Files

### `.team/{slug}/.state`

JSON — single source of truth for phase/loop state. Must be fsynced after every write.

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
  "artifacts": { ... },
  "artifact_manifests": [ ... ]
}
```

### `.team/index.md`

Project registry — master index of all team projects.

```
| Project | Status | Last Active | Mode | Topic Tags | Key Outcomes |
|---------|--------|-------------|------|------------|---------------|
```

### `.team/decisions.md`

Append-only decision log. Written on: IDEATION conclusion, BUILD start, DEPLOY gate.

---

## Output Schemas

### FormalBrief
```json
{ "title": "", "topic": "", "mode": "full|lite|debug|plan|pipeline", "objective": "", "scope": "", "constraints": [], "success_criteria": [], "assumptions": [] }
```

### Judgment
```json
{ "id": "uuid", "loop_number": 1, "timestamp": "ISO8601", "mode": "", "phase": "ideation|build|stress|security|deploy", "ultrathink_enabled": true|false, "reasoning_quality_assessment": { ... }, "agreements": [], "disagreements": [], "recommended_approach": "", "confidence": 0-100, "open_questions": [], "next_step": "continue|accept|stop" }
```

### FinalPlan
```json
{ "title": "", "objective": "", "recommended_approach": "", "deliverables": [], "timeline_weeks": 0, "owners": [], "risks_and_mitigations": [], "success_metrics": [], "next_actions": [] }
```

### StressReport
```json
{ "phase": "stress", "project": "", "timestamp": "ISO8601", "metrics": { "p99_latency_ms": { "value": 0, "threshold": 200, "passed": true|false }, "error_rate_percent": { ... }, "throughput_rps": { ... } }, "edge_cases": [], "overall_pass": true|false, "loop_count": 1 }
```

### SecurityClearance
```json
{ "phase": "security", "project": "", "timestamp": "ISO8601", "critical_cves": [], "sast_pass": true|false, "dast_pass": true|false, "dependency_audit": { "total": 0, "vulnerable": 0, "passed": true|false }, "overall_pass": true|false, "loop_count": 1 }
```

### DeployGate
```json
{ "phase": "deploy", "project": "", "timestamp": "ISO8601", "stress_clearance": { ... }, "security_clearance": { ... }, "artifact_integrity": { ... }, "overall_pass": true|false, "gate_decision": "APPROVED|REJECTED|ESCALATED", "rejection_reasons": [] }
```

---

## Directory Structure

```
.team/                          # Global team workspace (created on first /team call)
├── index.md                    # Project registry
├── decisions.md                # Append-only decision log
├── learnings.md                # Cross-session learnings (markdown)
├── pattern-library.md          # Aggregated patterns (auto-generated)
└── {project-slug}/             # Per-project directory
    ├── .state                  # Phase/loop state (JSON, fsynced)
    ├── .lock                   # Relay lock file
    ├── brief.md                # FormalBrief
    ├── plan.md                 # FinalPlan
    ├── judgments/              # Loop judgments (loop-1.json, loop-2.json)
    ├── build/                  # Build artifacts per agent
    │   ├── frontend/
    │   ├── backend/
    │   ├── devops/
    │   └── qa/
    ├── stress-report.md        # StressReport
    ├── security-clearance.md   # SecurityClearance
    ├── escalation/             # Escalation docs
    ├── session-memory.json     # Per-session memory capture
    ├── correction-log.md       # Enhanced CORRECTION entries
    └── .agent_outputs/        # Ephemeral agent output files (destroyed on session end)
```

---

## Error Types

| Error | Message | Handling |
|------|--------|----------|
| `StateTransitionError` | `Cannot enter ${targetPhase} from ${currentPhase}` | Caught by Orchestrator, written to `.state.errors[]` |
| `StateWriteError` | `Failed to write .state to disk — fsync verification failed` | Roll back state, release lock, exit gracefully |
| `ArtifactValidationError` | `Artifact manifest validation failed` | Fail phase immediately |
| `ManifestHashMismatchError` | `Artifact manifest_hash does not match actual content` | Reject artifact, fail BUILD |
| `ConcurrencyError` | `Team session already active for ${project}` | Reject new invocation |

---

## Known Limitations

1. **The skill is a prompt specification** — it defines agent prompts, state machines, and orchestration logic in YAML. It runs on Claude Code's built-in agent spawning infrastructure. There is no standalone binary or server.

2. **Relay lock is advisory** — the file-based lock (`.team/{slug}/.lock`) prevents concurrent `/team` invocations for the same project, but a rogue process could delete it. For multi-user environments, use a Redis/etcd lock as noted in `references/state_schema.md`.

3. **The BUILD phase prompts expect code generation** — Builder agents (`BuilderFrontend`, `BuilderBackend`, `BuilderDevOps`, `BuilderQA`) are defined as LLM prompts. Actual implementation depends on the LLM being able to write and verify code in your environment.

4. **STRESS/SECURITY agents need test infrastructure** — `LoadTester`, `EdgeCaseAnalyst`, `SecurityAuditor`, `DependencyScanner` require your project to have running servers, test suites, or CI pipelines to execute against. They will produce no-op results if the infrastructure is absent.

5. **Pre-flight memory query uses simple keyword matching** — ranking is based on tag overlap, recency, and `times_applied`. No semantic search.

6. **`/team rebuild-patterns` requires 3+ learnings per category** — categories with fewer than 3 learnings are not included in the pattern library.

---

## Design Principles

1. **Orchestrate, don't micromanage** — Wire patterns together; don't hand-hold every step.
2. **Debate before action** — Adversarial testing catches flawed reasoning before it becomes code.
3. **Phased execution with explicit gates** — Each phase gates the next; IDEATION cannot auto-advance to BUILD.
4. **Memory compounds over time** — Each session's corrections feed the next session's pre-flight query.
5. **Zero dead time** — Parallel agents, resumable checkpoints, and worktree isolation minimize idle time.
