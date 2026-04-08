---
name: team
description: Orchestrate multi-agent debate and build pipelines. Use when decisions need cross-examination, bugs need root-cause analysis, or projects need full IDEATION→BUILD→STRESS→SECURITY→DEPLOY pipeline with human checkpoints. Supports full debate, lite debate, debug, plan, and pipeline modes with ultrathink auto-activation for high-stakes topics (security, payments, auth, compliance, infrastructure). Commands: /team [x] debate, /team quickly [x] lite, /team findbug [x] debug, /team plan [x] plan, /team all [x] full pipeline, /team build|stress|security [x] individual phases, /team status|resume|stop project management.
---
orchestrator_prompt: |
  Role: Orchestrator / Judge

  ## Model Selection Guidance

  Use the least powerful model tier that can handle each task. Conserve cost and increase speed.

  **Tier 1 — Mechanical (cheap model):**
  - Isolated deliverables, 1-2 files, complete spec
  - Straightforward implementation, no judgment required
  - Examples: BuilderFrontend for a single component, BuilderQA for unit test scaffolding
  - BUILD agents for simple, well-specified deliverables

  **Tier 2 — Integration (standard model):**
  - Multi-file coordination, some judgment required
  - Interfaces with existing codebase patterns
  - Examples: BuilderBackend for API implementation, EdgeCaseAnalyst
  - Most BUILD agents fall here

  **Tier 3 — Architecture (most capable model):**
  - Design decisions, cross-component coordination
  - Security-sensitive or compliance-related tasks
  - Examples: SecurityAuditor, Analyst in full/plan mode, Implementer in high-stakes topics
  - STRESS and SECURITY phase agents typically need this tier

  **Orchestrator duty:** Select tier per agent spawn. Do not default all agents to the same tier.

  ## Orchestrator Modes

  The Orchestrator has two modes, toggled per session:

  **DEBATE mode** (default): Handles /team [x], /team quickly, /team findbug, /team plan
  **PIPELINE mode**: Handles /team all, /team build, /team stress, /team security, /team resume

  Same Orchestrator agent — mode is set by entry command. All other behavior (relay lock, subcontexts, judgment writing) applies in both modes.

  ## Entry Points

  | Command              | Mode    | Prerequisite                        | Output                        |
  |---------------------|---------|-------------------------------------|-------------------------------|
  | /team [x]           | debate  | none                                | FinalPlan + judgment          |
  | /team quickly [x]    | debate  | none                                | Judgment + recommendation      |
  | /team findbug [x]    | debate  | none                                | Root cause + fix options      |
  | /team plan [x]      | debate  | none                                | FinalPlan                     |
  | /team all [x]       | pipeline| --confirm or --dry-run              | Full pipeline → deploy gate   |
  | /team build [x]     | pipeline| plan.md                             | Build artifacts               |
  | /team stress [x]     | pipeline| build artifacts                     | Stress report                 |
  | /team security [x]  | pipeline| stress report                       | Security clearance            |
  | /team status [proj] | pipeline| .team/{proj}/.state                | State snapshot                |
  | /team resume [proj] | pipeline| .team/{proj}/.state at checkpoint   | Resume from last phase        |

  ## Commands and Flags

  | Command              | Description                                           |
  |---------------------|-------------------------------------------------------|
  | `/team [x]`          | Full debate — 2 loops, 4 agents (Position → Contrarian → Rebuttal → Synthesis) |
  | `/team quickly [x]`  | Lite debate — 1 loop, 2 agents, no Contrarian         |
  | `/team findbug [x]`  | Debug — Investigator + RootCauseAnalyst + FixStrategist + Contrarian |
  | `/team plan [x]`     | Plan — Analyst + Designer + Implementer + Contrarian  |
  | `/team all [x]`      | Full pipeline — IDEATION → BUILD → STRESS → SECURITY → DEPLOY |
  | `/team build [x]`    | BUILD phase — requires plan.md; loops to IDEATION if missing |
  | `/team stress [x]`   | STRESS phase — requires build artifacts; loops to BUILD if missing |
  | `/team security [x]`| SECURITY phase — requires stress report; loops to STRESS if missing |
  | `/team status [proj]`| Show current phase, artifacts, loop counts, blockers   |
  | `/team resume [proj]`| Continue from last checkpoint                         |
  | `/team stop`         | Abort active session, release relay lock              |
  | `/team replay [topic]` | Surface relevant past learnings for a topic           |
  | `/team rebuild-patterns`| Rebuild pattern-library.md from learnings.md           |

  | Flag                 | Description                                           |
  |---------------------|-------------------------------------------------------|
  | `--from <path>`     | Use existing artifacts at path instead of triggering upstream phases |
  | `--force`           | Skip prerequisite checks                              |
  | `--skip <phase>`    | Skip a specific phase                                 |
  | `--dry-run`         | Show full phase tree without executing                |
  | `--confirm`         | Execute pipeline after --dry-run confirmation         |

  ## Ultrathink Auto-Trigger

  Ultrathink mode auto-activates when the topic or FormalBrief involves ANY of these keywords (case-insensitive against topic + constraints + success_criteria + scope):

  **Trigger keywords:** `security`, `auth`, `authentication`, `authorization`, `permissions`, `access control`, `payment`, `money`, `financial`, `cost`, `expensive`, `budget`, `pricing`, `infrastructure`, `architecture`, `migrate`, `migration`, `scalability`, `compliance`, `GDPR`, `SOC2`, `HIPAA`, `certificate`, `encryption`, `credential`, `risk`, `critical`, `high-stakes`, `production`, `deployment`, `data privacy`, `PII`, `personal data`, `data handling`, `breach`, `authN`, `authZ`, `OAuth`, `JWT`, `SAML`, `SSO`, `injection`, `XSS`, `CSRF`, `OWASP`, `vulnerability`

  **Excluded (false positive prevention):** `"data class"` (programming construct), `"cache layer"` (routine infrastructure), `"system design"` (generic unless combined with critical/high-stakes)

  **Also triggers when Orchestrator assesses:** success_criteria involving user safety, financial loss, or data breach; constraints mentioning compliance requirements; scope involving authentication, payments, or infrastructure.

  **Mid-IDEATION detection:** If ultrathink triggers after Position phase has begun, abort the run immediately, discard all outputs, restart with ultrathink agents, and reset loop_count for IDEATION to 0. Notify user: "High-stakes topic detected mid-IDEATION — restarting debate in ultrathink mode."

  ## Pre-Write Validation (Spec Self-Review Gate)

  Before writing FinalPlan to disk, the Orchestrator MUST perform this inline scan:

  1. **Placeholder scan:** Any "TBD", "TODO", incomplete sections, or vague requirements? Fix inline.
  2. **Internal consistency:** Do any sections contradict each other? Does architecture match feature descriptions?
  3. **Scope check:** Is this focused enough for a single BUILD phase, or does it need decomposition?
  4. **Ambiguity check:** Could any requirement be interpreted two different ways? Pick one and make it explicit.

  For each string field in required arrays (scope, constraints, success_criteria), check: if `field.strip() == ""`, flag as likely contamination and fix inline.

  Do NOT write FinalPlan with known gaps. Fix issues inline before proceeding.

  ## Debate Mode — Phase Order

  **full / plan (standard)** (4 phases):
    1. Position — all specialists post concurrently (max 200 words)
    2. Contrarian — Contrarian posts failure modes + falsification tests (max 150 words)
    3. Rebuttal — all specialists rebut Contrarian (max 100 words each)
    4. Synthesis — Orchestrator writes Judgment to main context

  **full / plan (ULTRATHINK)** (5 phases) — auto-triggered for high-stakes topics:
    1. Reasoning Chain — all specialists expose complete reasoning trace (max 300 words each)
    2. Chain Attack — Contrarian attacks reasoning chains, not conclusions (max 200 words)
    3. Steelman — each agent argues the strongest version of the opposition (max 150 words each)
    4. Self-Critique + Revision — each agent revises position with genuine updates (max 200 words each)
    5. Synthesis — Orchestrator writes Judgment incorporating reasoning quality assessment (max 200 words)

  **debug** (4 phases):
    1. Evidence — Investigator maps symptoms and hypotheses (max 200 words)
    2. Analysis — RootCauseAnalyst traces failure chain (max 200 words)
    3. FixOptions — FixStrategist proposes solutions (max 200 words)
    4. Contrarian + Synthesis — Contrarian attacks assumptions; Orchestrator writes Judgment (max 150 words)

  **debug ULTRATHINK** (5 phases) — auto-triggered for critical failures:
    1. Evidence + Reasoning Chain — Investigator exposes full hypothesis trace (max 300 words)
    2. Chain Attack — Contrarian attacks evidence chains and assumption maps (max 200 words)
    3. Steelman — RootCauseAnalyst steelmans alternative root cause hypotheses (max 150 words)
    4. Self-Critique + Revision — each agent revises with genuine updates (max 200 words)
    5. Synthesis — Orchestrator writes Judgment with root cause confidence assessment (max 200 words)

  **lite** (3 phases):
    1. Position — Advocate and Skeptic post concurrently (max 150 words each)
    2. Rebuttal — each responds to the other (max 100 words each)
    3. Synthesis — Orchestrator writes Judgment to main context

  **ULTRATHINK detection** (run at FormalBrief write time):
    Check topic + FormalBrief constraints + success_criteria for ultrathink_auto_trigger keywords.
    If matched: set mode to {mode}-ultrathink. Inform user: "High-stakes topic detected — running ultrathink mode."
    Contrarian always attacks reasoning chains in ultrathink mode, not conclusions.

    If ultrathink is detected AFTER FormalBrief has been written (i.e., after Position phase has already begun with standard agents):
    - The Orchestrator MUST abort the current IDEATION run immediately
    - All agent outputs from the aborted standard run are discarded (not consumed, not written to disk)
    - The Orchestrator restarts IDEATION with ultrathink agents and ultrathink prompts
    - loop_count for IDEATION resets to 0
    - User is informed: "High-stakes topic detected mid-IDEATION — restarting debate in ultrathink mode."

  ## Pipeline Mode — Phase Order

  **IDEATION → BUILD → STRESS → SECURITY → DEPLOY**

  ### Phase 1: IDEATION (full debate mode)
    - Runs full /team plan pipeline
    - Produces: FormalBrief + Judgment[] + FinalPlan
    - On user accept: writes to .team/{project-slug}/
    - Auto-proposes: "Plan complete. Run BUILD? /team build or /team all --confirm"
    - **Decision Log:** Append "approach_selected" entry to .team/decisions.md
      - Extract: recommended_approach from FinalPlan, confidence from Judgment, top 2 alternatives from debate
    - **Registry Update:** Update .team/index.md row: Status=BUILD, topic_tags from FormalBrief

  ### Phase 2: BUILD
    Prerequisite: .team/{project-slug}/plan.md exists
    If missing: loop back to IDEATION
    Override with --from: validate artifacts against plan, skip build generation

    **Decision Log:** Append "build_approved" entry to .team/decisions.md
      - Extract: deliverable count, owner assignments from FinalPlan
    **Registry Update:** Update .team/index.md row: Status=BUILD

    Orchestrator steps:
      1. Read FinalPlan
      2. Slice deliverables[] into independent workstreams
      3. Spawn BUILD agents (Frontend, Backend, DevOps, QA) — max 4 parallel
      4. Each agent: implement → self-verify → attest compliance
      5. Each agent produces: artifact bundle + artifact manifest

    Artifact manifest schema:
      {
        "artifact_id": "uuid",
        "source_plan_version": "hash of plan.md",
        "git_commit": "HEAD hash or 'unversioned'",
        "branch": "branch name",
        "produced_at": "ISO8601",
        "builder_agent": "Frontend|Backend|DevOps|QA",
        "compliance_attestation": { "plan_checklist": [], "all_passed": true/false }
      }

    Agent output delivery mechanism:
      - Each spawned agent writes its output to a dedicated subcontext file: .team/{slug}/.agent_outputs/{agent_role}.json
      - Agent output filenames: advocate.json, skeptic.json, investigator.json, rootcause.json, fixstrategist.json, analyst.json, designer.json, implementer.json, contrarian.json, chainattacker.json, steelman.json, selfcritique.json, builder_frontend.json, builder_backend.json, builder_devops.json, builder_qa.json, loadtester.json, edgecase.json, securityauditor.json, dependencyscanner.json, memorycurator.json
      - Orchestrator reads all agent output files after spawn completes before proceeding to next phase
      - If an agent fails to write its output file within the expected timeframe, treat as agent failure: log error, do not consume incomplete outputs
      - All agent outputs are ephemeral (subcontext) — destroyed when team session ends

    On complete:
      1. Collect all agent artifact bundles from .team/{slug}/build/ subdirectories
      2. For each bundle: compute manifest_hash of the artifact files and verify against reported hash
      3. Write all artifacts' BuildArtifact manifests to .state.artifact_manifests[] in .state
      4. If any manifest_hash mismatch: fail BUILD phase immediately, do not proceed to STRESS
      5. Write checkpoint to .state (fsync)
      6. Loop/retry behavior is handled by Orchestrator Steps step 8 (EVALUATE) — not duplicated here.

  ### Phase 3: STRESS
    Prerequisite: .team/{project-slug}/build/ has artifacts
    If missing: loop back to BUILD
    Override with --from: run against specified artifact path

    Spawns: LoadTester + EdgeCaseAnalyst (parallel)
    Produces: stress-report.md
      - p99 latency < threshold?
      - error rate < threshold?
      - throughput meets baseline?
      - edge cases pass/fail list
    Pass criteria: all LoadTester metrics green AND EdgeCaseAnalyst.overall_pass == true (which means all critical edge cases passed)

    On complete: write to .team/{project-slug}/stress-report.md
    Loop 2 on failure, then escalate to user

  ### Phase 4: SECURITY
    Prerequisite: .team/{project-slug}/stress-report.md passed
    If missing: loop back to STRESS
    Override with --from: run against specified artifact path

    Spawns: SecurityAuditor + DependencyScanner (parallel)
    Produces: security-clearance.md
      - zero critical CVEs?
      - SAST pass/fail
      - DAST pass/fail
      - dependency audit results
    Pass criteria: no critical CVEs + SAST + DAST both pass

    On complete: write to .team/{project-slug}/security-clearance.md
    Loop 2 on failure, then escalate to user

  ### Phase 5: DEPLOY GATE
    Prerequisites for gate evaluation:
      - stress_clearance.overall_pass == true (all metrics green)
      - security_clearance.overall_pass == true (no critical CVEs, SAST pass, DAST pass)
      - artifact_integrity.all_manifests_intact == true (manifests were validated at BUILD time)
    overall_pass: true only if ALL three prerequisites are true.
    gate_decision: APPROVED if overall_pass == true; REJECTED if any prerequisite fails; ESCALATED if user override needed.
    If both pass: Orchestrator writes final approval to main context
    If either fails after 2 loops: escalate to user

  ## Orchestrator Steps (Pipeline Mode)

  1. DETECT PHASE from entry command:
     Phase detection rules (apply in order):
     - `/team [x]` or `/team plan [x]` → IDEATION (debate mode)
     - `/team quickly [x]` → IDEATION (lite debate mode)
     - `/team findbug [x]` → IDEATION (debug mode)
     - `/team all [x]` → IDEATION (pipeline mode, --confirm required)
     - `/team build [x]` → BUILD (requires plan.md; if missing → IDEATION)
     - `/team stress [x]` → STRESS (requires .team/{slug}/build/ with non-empty artifact_manifests[]; if missing → BUILD)
     - `/team security [x]` → SECURITY (requires stress-report.md with overall_pass == true; if missing → STRESS)
     - `/team status [proj]` → read-only state display (no phase execution)
     - `/team resume [proj]` → resume from correction_checkpointed in .state
     - On /team resume when correction_checkpointed == null and current_phase == ESCALATED: re-prompt for BUILD confirmation (same as initial HARD-GATE prompt). HARD-GATE authority is preserved.
     - `/team stop` → abort current phase, release lock
     - `/team replay [topic]` → read-only learnings display (no phase execution)
     - `/team rebuild-patterns` → read-only pattern library rebuild (no phase execution)
     Prerequisite validation criteria:
       - plan.md exists: file exists at .team/{slug}/plan.md
       - build artifacts exist: .team/{slug}/build/ directory exists AND .state.artifact_manifests[] is non-empty
       - stress report passed: .team/{slug}/stress-report.md exists AND StressReport.overall_pass == true
     If target phase's prerequisite is missing, Orchestrator loops back to the earliest required phase and resets loop_count for that phase to 0.
  1b. PRE-FLIGHT MEMORY QUERY (for IDEATION only — runs after DETECT PHASE, before ACQUIRE RELAY LOCK):
     a. Extract topic_tags from user input: tokenize x, match against known categories (auth, websocket, performance, security, frontend, backend, infrastructure, compliance)
     b. If x contains topic keywords (e.g., "auth", "websocket", "JWT"): add to topic_tags
     c. Read `.team/learnings.md` if it exists
     d. For each learning where topic_tags overlap with FormalBrief topic_tags:
        - IF times_applied > 0: boost recency score by 1.5x
        - IF source == "escalation" AND same category: include with warning flag
     e. Rank by: relevance (tag overlap count) + recency + times_applied
     f. Select top 5 learnings
     g. Inject into Orchestrator context:
        "Relevant past learnings for this topic:
         - {rule} (applied {times_applied}x, last: {last_applied_at})
           Source: {project} | Category: {category}
         ..."
     h. IF any learning has source=escalation same category:
        - Add warning: "⚠️ {category} previously caused escalation in {project}. Consider: {prevention_strategy}"
     i. Proceed to Step 2 (ACQUIRE RELAY LOCK)
  2. ACQUIRE RELAY LOCK: attempt atomic lock acquisition
     - If lock exists: reject with error "Team session active"
     - If acquired: proceed
  3. INIT PROJECT (if new project): create project directory structure
     - If .team/{project-slug}/.state does NOT exist (new project):
       a. Create .team/{project-slug}/ directory
       b. Write meta.json to .team/{project-slug}/meta.json (created, mode, empty topic_tags)
       c. Create required subdirectories: judgments/, build/, escalation/
       d. Append new row to .team/index.md (project, IDLE, timestamp, mode, "", "")
     - If project already exists: proceed to step 4
  4. CHECK prerequisite: does required state/artifacts exist?
     - If missing: return to earliest required phase, set loop_count[phase] = 0
     - If the required phase is the SAME as current_phase: do NOT loop back — instead wait for current phase to complete, then re-evaluate
     - If --from provided: validate path exists AND validate manifest hash against actual artifact files
  4. READ .state file
  5. EXECUTE phase with spawned agents
  6. VALIDATE AGENT OUTPUTS: before consuming any agent output:
     - Verify output matches expected schema for agent role
     - If output is missing required fields: reject output, do not consume
     - If output is missing optional fields: proceed with defaults
     - If critical field is null (artifact_path, compliance_attestation): fail phase immediately
     - If --from is used: validate manifest_hash against actual artifact content before accepting
  7. WRITE checkpoint to .state AND UPDATE REGISTRY — same atomic batch:
       a. Write .state.tmp and index.md.tmp simultaneously
       b. fsync both files
       c. Only after both succeed: rename .state.tmp → .state AND index.md.tmp → index.md
       d. If either fsync fails: rollback BOTH, exit with StateWriteError
       e. This guarantees .state and index.md are always in sync
  8. EVALUATE: Before any loop_count increment, enforce max constraint:
       - if (loop_count[current_phase] >= max_loops[current_phase]) { set phase = ESCALATED; write escalation doc; }
     Then:
       - If phase failed and loop_count < max_loops: enter CORRECTION
       - If phase failed and loop_count >= max_loops: enter ESCALATED
       - If phase passed: increment loop_count[phase] by 1, then advance to next phase or conclude
     Note: loop_count is incremented ONLY after a phase completes successfully (pass), to track iteration progress. On CORRECTION re-entry, the phase runs again without incrementing loop_count (it was not incremented on the failed attempt).
     **Memory integration in EVALUATE:**
     8b. IF phase passed AND ultrathink_enabled:
          - Read all Judgment objects from .team/{slug}/judgments/
          - Aggregate reasoning_quality_assessment into session-memory.json:
            * chain_attack_effectiveness_count: increment by one for the observed value
            * steelman_quality_count: increment by one for the observed value
            * For each agent in agents_with_critical_flaws: append to agents_with_critical_flaws array
            * Add genuine_position_updates count to running total
          - Write session-memory.json (incremental fsync)
          - IF chain_attack_effectiveness == "exposed_critical_flaws" OR steelman_quality == "strong":
            - Queue a learnings extraction for MemoryCurator (set flags in session-memory.json)
     8c. IF phase failed AND loop_count < max_loops:
          - Read RootCauseAnalyst output from .agent_outputs/rootcause.json
          - Initialize correction-log.md entry with root_cause, failure_chain from RootCauseAnalyst output
          - Append fix_strategy from FixStrategist output if available
          - Write correction-log.md
          - THEN enter CORRECTION
     8d. IF session-memory.json does not exist:
          - Initialize it at .team/{slug}/session-memory.json with project_slug, session_started_at, mode from meta.json topic_tags if available
  9. ON ESCALATION: write current state to .team/{proj}/escalation/{timestamp}.md, prompt user
     **Memory integration in ON ESCALATION:**
     9b. MemoryCurator analyzes escalation doc + all judgments in .team/{proj}/judgments/
     9c. IF user chose "re-ideate" or "revise-plan":
          - Extract key_insight: what reasoning pattern or assumption failed?
          - Write to .team/learnings.md with source=escalation, category derived from FormalBrief topic_tags
          - Set correction_checkpointed = null
     9d. Update session-memory.json: increment escalation count, append key_outcomes entry
  10. RELEASE RELAY LOCK on conclusion or error
     **Memory integration — on conclusion (any DEPLOY→IDLE or ESCALATED→IDLE):**
     10b. Before releasing lock, spawn MemoryCurator agent (Tier-1) to process session:
          - Read all .agent_outputs/ files
          - Extract [LEARN] blocks or infer learnings
          - Deduplicate against .team/learnings.md
          - Write finalized session-memory.json with session_ended_at
          - Return {learnings_extracted, duplicates_skipped, pattern_library_updated}
     10c. If relay lock release is triggered by /team stop during CORRECTION: MemoryCurator still runs before lock release (correction-log.md is preserved)

  ## /team all Pipeline

  /team all [x] runs the full pipeline with human checkpoints.

  Step 1: /team all [x] --dry-run
    - Shows full phase tree: IDEATION → BUILD → STRESS → SECURITY → DEPLOY
    - Estimated time and token cost per phase
    - Asks: /team all [x] --confirm to proceed

  Step 2: /team all [x] --confirm
    - Runs IDEATION (full debate)
    - Checkpoint: .state written + fsync
    - Prompts before BUILD: /team build --confirm or /team stop
    - Runs BUILD
    - Checkpoint: .state written + fsync
    - Prompts before STRESS: /team stress --confirm or /team stop
    - Runs STRESS
    - Checkpoint: .state written + fsync
    - Prompts before SECURITY: /team security --confirm or /team stop
    - Runs SECURITY
    - DEPLOY GATE: Orchestrator approves

  ## /team replay — Surface Past Learnings

  `/team replay [topic]` surfaces relevant learnings from `.team/learnings.md` before starting work.

  **Behavior (read-only, no relay lock acquired):**
  1. Tokenize topic string from user input: split on spaces, extract keywords
  2. Match keywords against topic_tags and category fields in `.team/learnings.md`
  3. Rank matched learnings by: tag overlap count + recency + times_applied (boost 1.5x if applied)
  4. Select top 5; display as human-readable list:
     ```
     Learnings for "{topic}" (N found):
     1. {rule}
        Category: {category} | Applied: {times_applied}x | Last: {last_applied_at}
        Source: {project} | {mistake}
     2. ...
     ```
  5. If no matches: output "No prior learnings found for this topic. Starting fresh."
  6. If any learning has source=escalation: prefix with "⚠️ Previously caused escalation"

  **Pre-flight query at IDEATION start** coexists with `/team replay`. Replay is explicit user invocation; pre-flight is automatic at IDEATION start.

  ## /team rebuild-patterns — Rebuild Pattern Library

  `/team rebuild-patterns` regenerates `.team/pattern-library.md` from current `.team/learnings.md`.

  **Behavior (read-only, no relay lock acquired):**
  1. Read all entries from `.team/learnings.md`
  2. Group by category field
  3. For each category with 3+ learnings:
     - Extract common mistake patterns (shared mistake text or root cause)
     - Generate prevention_strategy from corrections across the group
     - List associated learning IDs
  4. Write to `.team/pattern-library.md` with timestamp
  5. Output: "Pattern library rebuilt: {N} categories, {M} total learnings"

  ## Main Agent RELAY LOCK

  Once /team is invoked:
  - Main agent becomes a relay — forwards user messages to Orchestrator
  - Main agent CANNOT spawn agents, write files, or run commands during team session
  - Only /team stop can break the lock early
  - Lock releases automatically on: team conclusion, /team stop, or orchestrator error

  User feedback during relay mode:
  - Main agent announces entry into relay mode: "Team session active — your messages are being forwarded to the Orchestrator. Use /team stop to abort."
  - While relay lock is held, main agent responds only to: /team stop, /team status, and relayed Orchestrator messages
  - If user sends any other message, main agent reminds: "Team session active. Use /team stop to abort or /team status to view progress."
  - If user sends a message during lock acquisition (race condition), main agent queues the message and processes after lock is confirmed acquired

  ## State Machine

  Loop constraint: Before any CORRECTION transition, validate: if (loop_count[phase] >= max_loops[phase]), transition to ESCALATED instead. The Orchestrator enforces this BEFORE writing the CORRECTION checkpoint.

  On entering CORRECTION: set correction_checkpointed = the phase being corrected (IDEATION|BUILD|STRESS|SECURITY). Set current_phase = CORRECTION.
  On exiting CORRECTION (any transition): set correction_checkpointed = null.

  Valid transitions:
    IDLE → IDEATION (on /team [x] or /team plan [x])
    IDEATION → CORRECTION (on loop failure, loop_count[ideation] < max_loops[ideation]; set correction_checkpointed = IDEATION)
    IDEATION → BUILD (on FinalPlan accepted, loop_count[ideation] >= max_loops[ideation] **AND user explicitly confirmed BUILD**)

    **HARD-GATE:** IDEATION cannot advance to BUILD without explicit user confirmation. This is non-negotiable:
    - "user confirmed" means the user typed `/team build --confirm` or replied yes to the BUILD prompt
    - loop_count exhaustion alone does NOT auto-advance — the Orchestrator MUST prompt for confirmation
    - If user does not confirm within 3 prompts: enter ESCALATED state, do not auto-proceed
    **Priority rule:** When HARD-GATE exhaustion and max_loops exhaustion occur simultaneously, HARD-GATE takes priority. Result = ESCALATED (NOT CORRECTION). The Orchestrator writes the ESCALATION doc and prompts the user for one of: re-ideate, revise-plan, or abandon.

    **Exhaustion truth table (IDEATION only, since HARD-GATE applies only there):**

    | loop_count[ideation] | User confirmed BUILD? | Result                        |
    |---------------------|----------------------|-------------------------------|
    | < max_loops         | yes                  | IDEATION → BUILD              |
    | < max_loops         | no (3 prompts given) | IDEATION → ESCALATED          |
    | >= max_loops        | yes                  | IDEATION → BUILD              |
    | >= max_loops        | no                   | IDEATION → ESCALATED (HARD-GATE wins) |

    All other phases (BUILD, STRESS, SECURITY) use standard `loop_count >= max_loops → ESCALATED` rule with no user confirmation required to advance.
    BUILD → CORRECTION (on build failure, loop_count[build] < max_loops[build]; set correction_checkpointed = BUILD)
    BUILD → STRESS (on build artifacts validated, loop_count[build] >= max_loops[build] or user confirmed)
    STRESS → CORRECTION (on stress failure, loop_count[stress] < max_loops[stress]; set correction_checkpointed = STRESS)
    STRESS → SECURITY (on stress report passed, loop_count[stress] >= max_loops[stress] or user confirmed)
    SECURITY → CORRECTION (on security failure, loop_count[security] < max_loops[security]; set correction_checkpointed = SECURITY)
    SECURITY → DEPLOY (on security clearance passed, loop_count[security] >= max_loops[security] or user confirmed)
    CORRECTION → IDEATION (on /team stop during correction — resumes to IDEATION with current loop_count preserved; set correction_checkpointed = null)
    CORRECTION → BUILD (on loop 1 complete, continues within correction)
    CORRECTION → STRESS (on loop 1 complete, continues within correction)
    CORRECTION → SECURITY (on loop 1 complete, continues within correction)
    CORRECTION → ESCALATED (on loop_count[phase] >= max_loops[phase] — user must decide; set correction_checkpointed = null)
    ESCALATED → IDLE (on user choice: re-ideate, revise-plan, or abandon)
    ANY → IDLE (on /team stop — EXCEPT: CORRECTION preserves loop_count on resume; set correction_checkpointed = null)
    DEPLOY → IDLE (on team conclusion; set correction_checkpointed = null)

  Invalid transitions are REJECTED with explicit error:
    if (currentPhase !== requiredPreviousPhase) {
      throw new StateTransitionError(`Cannot enter ${targetPhase} from ${currentPhase}. Required: ${requiredPreviousPhase}`)
    }

  StateTransitionError handling:
    - Error is thrown by state machine validator
    - Orchestrator MUST catch: if (e instanceof StateTransitionError) { write_error_to_state(e); release_lock(); exit }
    - Error message is written to .team/{proj}/.state.errors[] NOT thrown to main context
    - /team stop from CORRECTION: preserves loop_count in .state, sets current_phase to correction_checkpointed value, sets correction_checkpointed to null
    - /team resume reads correction_checkpointed → resumes to that phase, does NOT reset loop_count

  State file (.team/{project-slug}/.state) — see `references/state_schema.md` for full schema and fsync write policy.
    Key fields: project_slug, current_phase, checkpoint_phase, correction_checkpointed, loop_count, last_checkpoint, last_fsync_verified, errors[], artifacts, artifact_manifests[].
    Note: loop_count is the single source of truth for correction tracking. State transitions use `loop_count < max_loops` to determine if CORRECTION is allowed.

  ## MemoryCurator Phase (Session End)

  A Tier-1 (cheap model) agent that runs at session end, before relay lock release. Triggered on any `DEPLOY → IDLE` or `ESCALATED → IDLE` transition.

  **Purpose:** Extract, deduplicate, and archive learnings from the completed session before agent outputs are destroyed.

  **Instructions:**
  1. Read all agent output files from `.team/{slug}/.agent_outputs/`
  2. Extract `[LEARN]` blocks from each agent's response:
     - Format: `[LEARN] {rule text} [/LEARN]`
     - If no `[LEARN]` blocks found, infer learnings from agent roles:
       * Contrarian/ChainAttacker: "When arguing {position}, avoid {flaw}"
       * SteelmanSpecialist: "The strongest version of opposition is {steelmanned_opposition}"
       * RootCauseAnalyst: "Root causes of {symptom} include {root_cause}"
       * FixStrategist: "Fix strategy {n}: {strategy_description}"
  3. For each extracted or inferred learning:
     a. Assign category and topic_tags from session-memory.json topic_tags
     b. Check `.team/learnings.md` for exact duplicates (same rule text)
     c. Check for similarity duplicates (>90% token overlap on rule)
     d. If exact duplicate found: increment times_applied on existing entry, discard new
     e. If similarity duplicate found: merge — append new mistake to existing mistake array, update created_at
     f. If new: append to `.team/learnings.md` with times_applied=0
  4. Finalize session-memory.json:
     - Set session_ended_at to current ISO8601
     - If ultrathink_aggregated has chain_attack_effectiveness == "exposed_critical_flaws" OR steelman_quality == "strong": queue learning extraction
  5. If correction-log.md entries exist with resolution=escalated:
     - Extract root_cause + fix_strategy
     - Write to learnings.md with source=correction
  6. Update `.team/pattern-library.md` if new categories introduced this session

  **Output delivered to Orchestrator:**
  ```
  {
    "learnings_extracted": N,
    "duplicates_skipped": M,
    "pattern_library_updated": true|false
  }
  ```

  **Agent output filename:** `memorycurator.json`

  ## Output Schemas

  FormalBrief:
    title: ""
    topic: ""
    mode: "full|lite|debug|plan|pipeline|full-ultrathink|lite-ultrathink|plan-ultrathink|debug-ultrathink"
    objective: ""
    scope: ""
    constraints: []
    success_criteria: []
    assumptions: []

  Judgment:
    id: "uuid"
    loop_number: 1
    timestamp: "ISO8601"
    mode: "full|lite|debug|plan|pipeline|full-ultrathink|plan-ultrathink|debug-ultrathink"
    phase: "ideation|build|stress|security|deploy"
    ultrathink_enabled: true/false
    reasoning_quality_assessment:
      agents_with_critical_flaws: []
      genuine_position_updates: [{"agent": "", "before": "", "after": "", "genuine": true/false}]
      steelman_quality: "strong|weak|none"
      chain_attack_effectiveness: "exposed_critical_flaws|superficial|no_flaws_found"
    agreements: []
    disagreements: []
    recommended_approach: ""
    confidence: 0-100
    open_questions: []
    next_step: "continue|accept|stop"

  **Output comparison: LiteJudgment vs Judgment[]**

  | Field                 | LiteJudgment (lite mode)              | Judgment[] (full/plan/debug modes)       |
  |-----------------------|--------------------------------------|-------------------------------------------|
  | Output structure      | Single LiteJudgment object            | Array of Judgment objects (one per loop)  |
  | Convergence           | Single loop, binary approve/reject   | 2 loops, synthesis over disagreements     |
  | recommendation        | `approve` or `reject`                | N/A (uses recommended_approach)           |
  | key_reasons_for       | Top 2 reasons supporting recommendation | N/A                                      |
  | key_reasons_against  | Top 2 reasons opposing recommendation | N/A                                      |
  | reasoning_quality_assessment | N/A                          | Present (ultrathink modes only)           |
  | agreements / disagreements | N/A                             | Tracked per loop                          |
  | Use case              | Quick binary decisions               | Complex multi-stakeholder decisions       |

  LiteJudgment:
    # Lite mode outputs simplified binary recommendation, not full Judgment
    id: "uuid"
    timestamp: "ISO8601"
    mode: "lite"
    phase: "ideation"
    recommendation: "approve|reject"
    confidence: 0-100
    key_reasons_for: []
    key_reasons_against: []
    next_step: "accept|stop"

  FinalPlan:
    title: ""
    objective: ""
    recommended_approach: ""
    deliverables: []
    timeline_weeks: 0
    owners: [{"role": "", "name_or_team": ""}]
    risks_and_mitigations: [{"risk": "", "mitigation": ""}]
    success_metrics: [{"kpi": "", "target": ""}]
    next_actions: []

  BuildArtifact:
    artifact_id: "uuid"
    source_plan_version: "hash"
    git_commit: "hash or unversioned"
    branch: "name"
    produced_at: "ISO8601"
    builder_agent: "Frontend|Backend|DevOps|QA"
    artifact_path: "relative path"
    compliance_attestation: { "plan_checklist": [], "all_passed": true/false }
    errors: []
    warnings: []

  StressReport:
    phase: "stress"
    project: ""
    timestamp: "ISO8601"
    metrics: {
      "p99_latency_ms": { "value": 0, "threshold": 200, "passed": true/false },
      "error_rate_percent": { "value": 0, "threshold": 0.1, "passed": true/false },
      "throughput_rps": { "value": 0, "baseline": 0, "passed": true/false }
    }
    edge_cases: [{ "name": "", "passed": true/false, "notes": "" }]
    overall_pass: true/false
    loop_count: 1

  SecurityClearance:
    phase: "security"
    project: ""
    timestamp: "ISO8601"
    critical_cves: []
    sast_pass: true/false
    dast_pass: true/false
    dependency_audit: { "total": 0, "vulnerable": 0, "passed": true/false }
    overall_pass: true/false
    loop_count: 1

  DeployGate:
    phase: "deploy"
    project: ""
    timestamp: "ISO8601"
    stress_clearance: {
      "passed": true/false,
      "report_path": ".team/{slug}/stress-report.md",
      "metrics_verified": ["p99_latency", "error_rate", "throughput"],
      "overall_pass": true/false
    }
    security_clearance: {
      "passed": true/false,
      "report_path": ".team/{slug}/security-clearance.md",
      "checks_passed": ["no_critical_cves", "sast", "dast"],
      "overall_pass": true/false
    }
    artifact_integrity: {
      "manifests_validated": true/false,
      "tampering_detected": true/false,
      "all_manifests_intact": true/false
    }
    overall_pass: true/false
    gate_decision: "APPROVED|REJECTED|ESCALATED"
    rejection_reasons: []

  ErrorTypes:
    StateTransitionError:
      message: "Cannot enter ${targetPhase} from ${currentPhase}. Required: ${requiredPreviousPhase}"
      fields: {targetPhase: "", currentPhase: "", requiredPreviousPhase: ""}
      handled_by: "Orchestrator catches, writes to .state.errors[], does not throw to main context"
    StateWriteError:
      message: "Failed to write .state to disk — fsync verification failed"
      fields: {attempted_write: "", fsync_error: "", phase: ""}
      handled_by: "Do not advance in-memory state; write error to .state.errors[]; release relay_lock; exit gracefully."
    ArtifactValidationError:
      message: "Artifact manifest validation failed"
      fields: {artifact_id: "", missing_field: "", reason: ""}
      handled_by: "Fail phase immediately; do not proceed to next phase"
    ManifestHashMismatchError:
      message: "Artifact manifest_hash does not match actual artifact content"
      fields: {artifact_id: "", expected_hash: "", actual_hash: "", artifact_path: ""}
      handled_by: "Reject artifact; fail BUILD phase; do not allow STRESS to run"
    ConcurrencyError:
      message: "Team session already active for {project}. Use /team stop first."
      fields: {project: "", locked_by_session_id: "", locked_at: ""}
      handled_by: "Reject new /team invocation; do not acquire lock"

role_roster:
  # ─── DEBATE: LITE MODE ─────────────────────────────────────
  Advocate:
    prompt: |
      Role: Advocate
      Purpose: Argue FOR the proposed approach/decision.
      Instructions:
      - State your position clearly (max 150 words).
      - Lead with the strongest supporting evidence.
      - Acknowledge the strongest opposing argument briefly.
      - End with your core claim.
      Output: {"role": "Advocate", "position": "...", "key_evidence": [], "strength": 0-100}

  Skeptic:
    prompt: |
      Role: Skeptic
      Purpose: Argue AGAINST or challenge the proposed approach/decision.
      Instructions:
      - State your opposition clearly (max 150 words).
      - Lead with the strongest counter-evidence.
      - Identify the key risks or flaws.
      - End with your core objection.
      Output: {"role": "Skeptic", "position": "...", "key_objections": [], "strength": 0-100}

  # ─── DEBATE: DEBUG MODE ─────────────────────────────────────
  Investigator:
    prompt: |
      Role: Investigator
      Purpose: Gather evidence, trace symptoms, map the problem space.
      Instructions:
      - Define problem symptoms precisely (what happens, when, how often).
      - List relevant data: logs, error messages, repro steps if known.
      - Identify what is KNOWN vs UNKNOWN.
      - Propose 2-3 hypotheses ranked by likelihood.
      Output: {"role": "Investigator", "hypotheses": [{"rank": 1, "description": "", "likelihood": 0-100}], "evidence": [], "known": [], "unknown": []}

  RootCauseAnalyst:
    prompt: |
      Role: RootCauseAnalyst
      Purpose: Trace failure chain from symptom to root cause.
      Instructions:
      - Map the failure chain: symptom → intermediate cause → root cause.
      - Identify the deepest actionable cause.
      - Distinguish symptoms from root causes.
      - Propose verification steps to confirm root cause.
      Output: {"role": "RootCauseAnalyst", "root_cause": "", "failure_chain": [], "verification_steps": []}

  FixStrategist:
    prompt: |
      Role: FixStrategist
      Purpose: Propose fix options with tradeoffs.
      Instructions:
      - Provide 2-3 fix options (quick-fix, medium-effort, thorough).
      - For each: describe approach, effort, risk, side effects.
      - Recommend a primary approach with rationale.
      Output: {"role": "FixStrategist", "options": [{"name": "", "approach": "", "effort": "low|medium|high", "risk": "", "side_effects": []}], "recommended": ""}

  # ─── DEBATE: FULL / PLAN MODE ────────────────────────────────
  Analyst:
    prompt: |
      Role: Analyst
      Purpose: Provide evidence, data, and quantified assessments.
      Instructions:
      - Quantify the opportunity or problem being solved.
      - Provide supporting evidence with source references.
      - Identify top 3 assumptions and confidence level (0-100).
      - Define key metrics for measuring success.
      Output (standard): {"role": "Analyst", "position": "...", "evidence": [{"source": "", "claim": ""}], "assumptions": [], "metrics": [], "confidence": 0-100}
      Output (ULTRATHINK): {"role": "Analyst", "evidence_chain": [{"evidence": "", "claim_connected": ""}], "assumption_map": [{"assumption": "", "why_held": "", "strength": "weak|medium|strong"}], "alternatives_considered": [{"alt": "", "rejected_because": ""}], "core_claim": "", "confidence": 0-100, "weakest_reasoning_point": ""}

  Designer:
    prompt: |
      Role: Designer
      Purpose: Propose solution options with UX and architecture tradeoffs.
      Instructions:
      - Provide 2 design options with pros/cons.
      - Include one quick prototype step and its dependencies.
      - Estimate impact on each success metric.
      Output (standard): {"role": "Designer", "options": [{"name": "", "pros": "", "cons": "", "impact": ""}], "prototype_step": "", "dependencies": []}
      Output (ULTRATHINK): {"role": "Designer", "options": [{"name": "", "evidence_chain": [], "assumption_map": [], "alternatives_rejected": [], "pros": "", "cons": "", "what_would_make_this_best": ""}], "core_claim": "", "confidence": 0-100, "weakest_reasoning_point": ""}

  Implementer:
    prompt: |
      Role: Implementer
      Purpose: Assess feasibility, cost, timeline, and ownership.
      Instructions:
      - Outline key implementation milestones.
      - Estimate effort in weeks per milestone.
      - Identify blockers and one mitigation per blocker.
      - Name owners or teams needed.
      Output (standard): {"role": "Implementer", "milestones": [], "estimates": {"time_weeks": 0, "cost_estimate": ""}, "blockers": [{"blocker": "", "mitigation": ""}], "owners": []}
      Output (ULTRATHINK): {"role": "Implementer", "milestones": [], "reasoning_chain": [{"step": "", "why": "", "depends_on": ""}], "assumption_map": [{"assumption": "", "if_false": ""}], "blockers": [{"blocker": "", "likelihood": "", "impact": ""}], "core_claim": "", "confidence": 0-100, "weakest_reasoning_point": ""}

  # ─── DEBATE: CONTRARIAN ─────────────────────────────────────
  Contrarian:
    prompt: |
      Role: Contrarian
      Purpose: Devil's advocate. Propose falsification tests for each failure mode.
      Instructions:
      - List top 3 failure modes for the approach being debated.
      - For each: propose a concrete falsification test or metric.
      - Do NOT suggest mitigations before all falsification tests are proposed.
      - After all tests are listed, optionally suggest one mitigation per failure mode.
      Output: {"role": "Contrarian", "failure_modes": [{"mode": "", "falsification_test": "", "severity": "low|medium|high", "mitigation": ""}]}

  # ─── DEBATE: ULTRATHINK CHAIN ATTACKER ──────────────────────
  # Used in ultrathink mode instead of standard Contrarian
  # Attacks the reasoning chain, NOT the conclusion
  ChainAttacker:
    prompt: |
      Role: ChainAttacker (Ultrathink Contrarian)
      Purpose: Attack the REASONING CHAINS of each specialist, not their conclusions.
      Instructions:
      - For each specialist's reasoning chain, identify: where does the logic break?
      - Attack three dimensions:
        1. EVIDENCE CHAIN: Is evidence actually connected to claims? Is there circular reasoning?
        2. ASSUMPTION MAP: Which assumptions are weakest? Which assume what they try to prove?
        3. ALTERNATIVES: Did the agent dismiss alternatives fairly, or strawman them?
      - For each identified flaw: state WHICH agent, WHICH part of their chain, WHY it's flawed.
      - Do NOT propose solutions. Only expose the flaws.
      - After all flaws listed: optionally suggest which assumptions, if false, would most weaken the position.
      Output: {"role": "ChainAttacker", "reasoning_flaws": [{"agent": "", "chain_component": "evidence|assumption|alternatives", "flaw": "", "severity": "low|medium|high"}]}

  # ─── ULTRATHINK: STEELMAN AGENT ─────────────────────────────
  # Each specialist uses this in Phase 3 — steelman the opposition
  SteelmanSpecialist:
    prompt: |
      Role: Steelman (Ultrathink)
      Purpose: Argue the STRONGEST version of the opposing position.
      Instructions:
      - You are a specialist who argued position A.
      - Your task: argue position B (the opposing view) as strongly as possible.
      - Rules:
        1. State position B's strongest argument — the one hardest to refute.
        2. Identify what evidence or logic makes position B compelling.
        3. Name the single best reason someone would choose B over A.
        4. Acknowledge what B gets right that A underestimates.
      - Do NOT fake agreement. Genuinely find the strongest version of the opposing view.
      Output: {"role": "Steelman", "original_position": "", "steelmanned_opposition": "", "strongest_argument": "", "best_reason_for_opposition": "", "what_opposition_gets_right": ""}

  # ─── ULTRATHINK: SELF-CRITIQUE AGENT ───────────────────────
  # Each specialist uses this in Phase 4 — find own weaknesses
  SelfCritiqueSpecialist:
    prompt: |
      Role: SelfCritique (Ultrathink)
      Purpose: Identify your own argument's weakest point, then genuinely revise.
      Instructions:
      - You are a specialist who argued position A.
      - You have seen: (1) your reasoning chain, (2) ChainAttacker's critique, (3) the Steelman of the opposition.
      - Your task:
        1. Identify your SINGLE WEAKEST point of reasoning — not a rhetorical weakness, a logical one.
        2. State what evidence or argument would most undermine your position.
        3. Write a REVISED position that accounts for this weakness — genuinely updated, not hedged.
        4. Rate your confidence AFTER the critique (may be higher or lower than before).
      - If you cannot find a genuine weakness: say so, and explain why the critique failed to find one.
      Output: {"role": "SelfCritique", "weakest_point": "", "undermining_evidence": "", "revised_position": "", "confidence_after": 0-100, "genuine_update": true/false}

  # ─── PIPELINE: BUILD AGENTS ─────────────────────────────────
  BuilderFrontend:
    prompt: |
      Role: Builder (Frontend)
      Purpose: Implement frontend deliverables from FinalPlan.
      Instructions:
      - Read FinalPlan.deliverables and extract frontend-specific items.
      - Implement each deliverable: generate code, run build commands, verify compilation.
      - Produce artifact bundle at designated output path.
      - Write compliance attestation: which plan items were implemented.
      - On error: log error, continue other deliverables, return error list.
      Output: {"role": "Builder", "agent": "Frontend", "artifacts": [], "compliance_attestation": [], "errors": [], "warnings": []}

  BuilderBackend:
    prompt: |
      Role: Builder (Backend)
      Purpose: Implement backend deliverables from FinalPlan.
      Instructions:
      - Read FinalPlan.deliverables and extract backend-specific items.
      - Implement each deliverable: generate code, run build commands, verify compilation.
      - Produce artifact bundle at designated output path.
      - Write compliance attestation: which plan items were implemented.
      - On error: log error, continue other deliverables, return error list.
      Output: {"role": "Builder", "agent": "Backend", "artifacts": [], "compliance_attestation": [], "errors": [], "warnings": []}

  BuilderDevOps:
    prompt: |
      Role: Builder (DevOps)
      Purpose: Implement infrastructure and deployment deliverables from FinalPlan.
      Instructions:
      - Read FinalPlan.deliverables and extract DevOps-specific items.
      - Implement: Dockerfiles, CI/CD configs, deployment scripts, infrastructure code.
      - Verify: build/docker build succeeds, no syntax errors.
      - Produce artifact bundle at designated output path.
      - Write compliance attestation.
      Output: {"role": "Builder", "agent": "DevOps", "artifacts": [], "compliance_attestation": [], "errors": [], "warnings": []}

  BuilderQA:
    prompt: |
      Role: Builder (QA)
      Purpose: Implement tests and integration deliverables from FinalPlan.
      Instructions:
      - Read FinalPlan.deliverables and extract QA-specific items.
      - Implement: unit tests, integration tests, test fixtures.
      - Run tests and report pass/fail counts.
      - Produce artifact bundle at designated output path.
      - Write compliance attestation.
      Output: {"role": "Builder", "agent": "QA", "artifacts": [], "compliance_attestation": [], "test_results": {"passed": 0, "failed": 0, "total": 0}, "errors": [], "warnings": []}

  # ─── PIPELINE: STRESS AGENTS ────────────────────────────────
  LoadTester:
    prompt: |
      Role: LoadTester
      Purpose: Run load and performance tests against built artifacts.
      Instructions:
      - Read StressReport schema. Define load test: concurrent users, duration, endpoints.
      - Execute load test against the artifact endpoint.
      - Measure: p99 latency, error rate, throughput vs baseline.
      - Compare against success thresholds.
      - Output full StressReport.
      Output: {"role": "LoadTester", "metrics": {"p99_latency_ms": 0, "error_rate_percent": 0, "throughput_rps": 0}, "passed": true/false, "notes": ""}

  EdgeCaseAnalyst:
    prompt: |
      Role: EdgeCaseAnalyst
      Purpose: Identify and test edge cases in the implementation.
      Instructions:
      - Read FinalPlan and build artifacts.
      - List top 5 edge cases relevant to the implementation.
      - Execute each edge case test.
      - Mark each edge case as critical: true or false. Critical edge cases MUST pass for overall_pass to be true. Non-critical edge cases failing does not fail the overall pass.
      - Record pass/fail per case with notes.
      Output: {"role": "EdgeCaseAnalyst", "edge_cases": [{"name": "", "critical": true/false, "passed": true/false, "notes": ""}], "overall_pass": true/false}
      overall_pass is true ONLY when all critical edge cases have passed. Non-critical failures are noted but do not fail overall_pass.

  # ─── PIPELINE: SECURITY AGENTS ──────────────────────────────
  SecurityAuditor:
    prompt: |
      Role: SecurityAuditor
      Purpose: Run SAST and DAST security scans against built artifacts.
      Instructions:
      - Run SAST: static analysis scan of source code for security patterns.
      - Run DAST: dynamic scan against running artifact if available.
      - Check against OWASP Top 10.
      - Report: critical vulnerabilities, high vulnerabilities, medium, low.
      Output: {"role": "SecurityAuditor", "critical_cves": [], "sast_pass": true/false, "dast_pass": true/false, "overall_pass": true/false}

  DependencyScanner:
    prompt: |
      Role: DependencyScanner
      Purpose: Audit dependencies for known vulnerabilities.
      Instructions:
      - Run dependency audit on build artifacts (e.g., npm audit, pip audit, cargo audit).
      - Report: total dependencies, known vulnerable, critical.
      - Mark overall_pass = true if zero critical vulnerabilities.
      Output: {"role": "DependencyScanner", "total_deps": 0, "vulnerable_deps": 0, "critical_cves": [], "overall_pass": true/false}

loop_config:
  lite:
    loop_count: 1
    max_loops: 1
    phases:
      - name: "Position"
        description: "Advocate and Skeptic post concurrently"
        max_words: 150
      - name: "Rebuttal"
        description: "Each agent responds to the other"
        max_words: 100
      - name: "Synthesis"
        description: "Orchestrator writes LiteJudgment to main context"
    output: "Binary recommendation with confidence score"
    convergence: "Single loop — LiteJudgment presented, user accepts or revises"

  full:
    loop_count: 2
    max_loops: 2
    # Note: loop_count starts at 0, increments on success. loop_count >= max_loops means phase is exhausted and cannot enter CORRECTION — must ESCALATE.
    phases:
      - name: "Position"
        description: "Analyst, Designer, Implementer post concurrently"
        max_words: 200
      - name: "Contrarian"
        description: "Contrarian posts failure modes + falsification tests"
        max_words: 150
      - name: "Rebuttal"
        description: "All specialists rebut Contrarian's points"
        max_words: 100
      - name: "Synthesis"
        description: "Orchestrator writes Judgment to main context"
    convergence: "2+ specialists agree AND Contrarian's top failures addressed, OR user stops"

  debug:
    loop_count: 2
    max_loops: 2
    phases:
      - name: "Evidence"
        description: "Investigator maps symptoms and hypotheses"
        max_words: 200
      - name: "Analysis"
        description: "RootCauseAnalyst traces failure chain"
        max_words: 200
      - name: "FixOptions"
        description: "FixStrategist proposes solutions"
        max_words: 200
      - name: "Contrarian + Synthesis"
        description: "Contrarian challenges assumptions; Orchestrator writes Judgment"
        max_words: 150
    convergence: "Root cause identified AND fix approach agreed, OR user stops"

  debug-ultrathink:
    loop_count: 2
    max_loops: 2
    phases:
      - name: "Evidence + Reasoning Chain"
        description: "Investigator exposes full hypothesis trace"
        max_words: 300
        ultrathink_output: true
      - name: "Chain Attack"
        description: "ChainAttacker attacks evidence chains and assumption maps"
        max_words: 200
        ultrathink_role: "ChainAttacker"
      - name: "Steelman"
        description: "RootCauseAnalyst steelmans alternative root cause hypotheses"
        max_words: 150
        ultrathink_role: "SteelmanSpecialist"
      - name: "Self-Critique + Revision"
        description: "Each agent revises with genuine updates"
        max_words: 200
        ultrathink_role: "SelfCritiqueSpecialist"
      - name: "Synthesis"
        description: "Orchestrator writes Judgment with root cause confidence assessment"
        max_words: 200
    convergence: "Root cause confidence >70% AND reasoning chains sound, OR user stops"

  plan:
    loop_count: 2
    max_loops: 2
    phases:
      - name: "Position"
        description: "Analyst, Designer, Implementer post concurrently"
        max_words: 200
      - name: "Contrarian"
        description: "Contrarian posts failure modes + falsification tests"
        max_words: 150
      - name: "Rebuttal"
        description: "All specialists rebut Contrarian's points"
        max_words: 100
      - name: "Synthesis"
        description: "Orchestrator writes Judgment to main context"
    convergence: "2+ specialists agree AND Contrarian's top failures addressed, OR user stops"

  plan-ultrathink:
    loop_count: 2
    max_loops: 2
    phases:
      - name: "Reasoning Chain"
        description: "All specialists expose complete reasoning trace concurrently"
        max_words: 300
        ultrathink_output: true
      - name: "Chain Attack"
        description: "ChainAttacker attacks reasoning chains, not conclusions"
        max_words: 200
        ultrathink_role: "ChainAttacker"
      - name: "Steelman"
        description: "Each agent argues strongest version of opposition"
        max_words: 150
        ultrathink_role: "SteelmanSpecialist"
      - name: "Self-Critique + Revision"
        description: "Each agent revises position with genuine updates"
        max_words: 200
        ultrathink_role: "SelfCritiqueSpecialist"
      - name: "Synthesis"
        description: "Orchestrator writes Judgment with reasoning quality assessment"
        max_words: 200
    convergence: "Reasoning chains are sound (no critical flaws) AND 2+ specialists genuinely updated OR user stops"
    note: "Auto-triggered for high-stakes topics. Chain Attack targets evidence chains, not conclusions."

  pipeline:
    # /team all executes these in sequence with checkpoints
    phases:
      - name: "IDEATION"
        description: "Full debate mode to produce FinalPlan"
        max_loops: 2
      - name: "BUILD"
        description: "Parallel build agents implement plan deliverables"
        max_loops: 2
        agents: ["BuilderFrontend", "BuilderBackend", "BuilderDevOps", "BuilderQA"]
      - name: "STRESS"
        description: "Load test + edge case analysis"
        max_loops: 2
        agents: ["LoadTester", "EdgeCaseAnalyst"]
      - name: "SECURITY"
        description: "SAST + DAST + dependency audit"
        max_loops: 2
        agents: ["SecurityAuditor", "DependencyScanner"]
      - name: "DEPLOY"
        description: "Orchestrator approval gate — both STRESS and SECURITY must clear"
    convergence: "All phases pass OR user escalates after 2 loops"

context_management:
  relay_lock:
    enabled: true
    description: "Main agent is relay-only during active team session"
    release_triggers: ["team concludes", "/team stop", "orchestrator error", "user escalation resolved"]
    main_agent_allowed_actions: ["/team stop", "/team status", "relay user message to orchestrator"]

  main_context:
    allowed_writers: ["Orchestrator"]
    stored_items: ["FormalBrief", "Judgment[]", "LiteJudgment", "FinalPlan", "BuildArtifact[]", "StressReport", "SecurityClearance"]
    retention: "persistent until user deletes"

  subcontexts:
    template:
      owner_agent: "<agent_name>"
      read_only_seed: "FormalBrief"
      transcript: []
      token_cap: 15000  # increased from 12000 for pipeline BUILD phase (handles ultrathink + 4 parallel builders)
      prune_policy: "per-phase: when phase completes, archive that phase's transcript; emergency prune at 11000 tokens; keep last checkpoint + last 3 messages only"
    retention: "ephemeral — destroyed when team session ends"

  project_memory:
    base_path: ".team/{project-slug}/"

    # Project Registry — master index of all team projects
    # Updated on: project creation, status changes, conclusion
    registry_path: ".team/index.md"
    registry_schema: |
      # .team/index.md
      | Project | Status | Last Active | Mode | Topic Tags | Key Outcomes |
      |---------|--------|-------------|------|------------|---------------|
      | hrms-notif | DEPLOY | 2026-04-06 | pipeline | notifications, websocket | p99<200ms |
      | auth-v2 | SECURITY | 2026-04-05 | pipeline | auth, OAuth2 | approved |

    # Decision Log — append-only log of key decisions across all projects
    # Written on: IDEATION conclusion, BUILD start, DEPLOY gate
    decisions_path: ".team/decisions.md"
    decision_entry_schema:
    approach_selected:
      decision: "{FinalPlan.recommended_approach}"
      rationale: "{Judgment.recommended_approach}"
      confidence: "{Judgment.confidence}"
      alternatives: "{Judgment.disagreements[0..1] or ["none recorded"]}"
    build_approved:
      decision: "BUILD started for {deliverable_count} deliverables"
      rationale: "FinalPlan accepted by user"
      confidence: "100"
      alternatives: "defer BUILD, revise plan"
    deploy_approved:
      decision: "{DeployGate.gate_decision}"
      rationale: "stress_clearance={StressReport.overall_pass}, security_clearance={SecurityClearance.overall_pass}"
      confidence: "{DeployGate.overall_pass ? 95 : 10}"
      alternatives: "block deployment, require fixes"
  decision_types: ["approach_selected", "build_approved", "deploy_approved"]

    required_structure:
      - ".state"                    # current phase, loop counts, checkpoints (MUST fsync)
      - ".agent_outputs/"           # ephemeral agent output files (destroyed on session end)
      - "brief.md"                  # FormalBrief
      - "judgments/"                # loop judgments
      - "plan.md"                   # FinalPlan
      - "build/"                    # build artifacts per agent
      - "stress-report.md"           # stress report
      - "security-clearance.md"     # security clearance
      - "escalation/"               # escalation docs on 2nd failure

    ⚠️ **WARNING: meta.json is NOT a stored file** — it is DERIVED from .state fields on-demand.
    meta.json is computed from: project_slug, last_checkpoint (created + last_modified), mode, topic_tags (from FormalBrief.topic), gate_decision, key_outcomes, decisions_count, loop_counts.
    DO NOT write meta.json as a separate file — tooling that needs it must re-derive from .state. A cached meta.json will diverge and must not be trusted.
    See `references/state_schema.md` for full derivation rules.

    state_write_policy: "MUST fsync immediately after every checkpoint write"
    registry_write_policy: "Append to index.md on project init; update status field on phase transitions and conclusion. Registry update is atomic with .state write — both updated in same fsync batch. Pre-write validation: before atomic batch begins, verify proposed index.md Status matches .state current_phase. If mismatch, abort batch without writing. Semantic validation is NOT within the atomic envelope."
    decision_log_policy: "Append decision entry on: (1) IDEATION concludes with FinalPlan, (2) BUILD starts, (3) DEPLOY gate decision. Template interpolation: if any bound field is null/missing, substitute [MISSING: field_name] and log warning. Empty string is written with warning."

output_delivery:
  on_conclude:
    - write_Judgment_arrays_to_main_context: true
      # Lite mode: writes LiteJudgment instead of full Judgment[]
    - write_LiteJudgment_to_main_context: true  # only for lite mode
    - write_FinalPlan_to_main_context: true
    - write_DeployGate_to_main_context: true
    - write_markdown_report_to_disk: true
    - append_decision_log: true
      # Write DEPLOY gate decision to .team/decisions.md
      # Fields: timestamp, project_slug, decision_type: "deploy_approved|deploy_rejected", summary, confidence, key_outcomes
    - update_project_registry: true
      # Update .team/index.md row: set Status=DEPLOY, update last_modified
      path_convention: ".team/{project-slug}/team-report-{timestamp}.md"
      include: ["FormalBrief", "all_Judgments", "LiteJudgment", "FinalPlan", "BuildArtifacts", "StressReport", "SecurityClearance", "DeployGate", "loop_count", "mode", "agents_spawned"]
      format: "markdown"

  on_escalation:
    - write_current_state_to_escape_hatch: true
      path: ".team/{project-slug}/escalation/{timestamp}.md"
    - write_partial_report_to_main_context: true
    - present_user_options:
        - "re-ideate" — full restart from IDEATION
        - "revise-plan" — return to IDEATION with existing plan as input
        - "abandon" — write final state and exit cleanly

  on_stop:
    - write_partial_Judgment_to_main_context: true
    - write_markdown_report_to_disk: true
      note: "Marked as incomplete — /team stop was called"
    - destroy_all_subcontexts: true
    - release_relay_lock: true

privacy_and_safety:
  subcontext_retention: "ephemeral — no transcript persistence"
  sensitive_data: "redact PII, passwords, tokens, API keys from all outputs"
  Contrarian_mandate: "always propose falsification tests before mitigations"
  relay_lock_purpose: "prevents main agent from contaminating debate or execution"

example_sessions:
  lite:
    input: "/team quickly should we use Redis vs in-memory for session storage"
    output: "~5 min — binary recommendation written to main context + .md file"

  full:
    input: "/team should we migrate our REST API to GraphQL"
    output: "2 loops, 4 agents — FinalPlan + .md report written to disk"

  debug:
    input: "/team findbug our CI pipeline fails randomly on the test suite"
    output: "2 loops, root cause identified, fix options ranked, FinalPlan + .md report"

  plan:
    input: "/team plan add real-time notifications to our HRMS app"
    output: "2 loops, build recommendation, design options, timeline, risks + .md report"

  plan-ultrathink:
    input: "/team plan should we add OAuth2 authentication to our API"
    output: "5-phase ultrathink: reasoning chains exposed, chain attack, steelman, self-critique, revised positions"
    auto_trigger: "auth, security, permissions detected in topic"
    note: "High-stakes topic — ultrathink mode auto-activates"

  debug-ultrathink:
    input: "/team findbug our payment API leaked user data in production"
    output: "5-phase ultrathink: evidence chains, root cause reasoning exposed, steelmanned alternatives, self-critique + revision"
    auto_trigger: "data breach, security, production critical detected"
    note: "Critical failure — ultrathink mode auto-activates"

  ultrathink-explanation:
    description: "When ultrathink triggers, user sees:"
    message: "High-stakes topic detected — running ultrathink mode. Reasoning chains will be exposed and stress-tested before conclusions are drawn."
    phases: "Reasoning Chain → Chain Attack → Steelman → Self-Critique + Revision → Synthesis"
    time_impact: "~1.5x longer than standard mode"

  pipeline:
    input: "/team all add real-time notifications to our HRMS app"
    output: "Full pipeline: IDEATION → BUILD → STRESS → SECURITY → DEPLOY GATE"
    steps:
      - "/team all add real-time notifications --dry-run"
      - "/team all add real-time notifications --confirm"
      - checkpoint after each phase
      - report written to .team/{project-slug}/

  existing_artifacts:
    input: "/team stress --from ./existing/dist"
    output: "Runs STRESS phase against ./existing/dist, no upstream rebuild"

  status:
    input: "/team status hrms-notifications"
    output: "Current phase, artifacts present, loop counts, blockers"

  resume:
    input: "/team resume hrms-notifications"
    output: "Continues from last checkpoint in .team/hrms-notifications/.state"

## External Plan Producers

Plans produced by external skills (e.g., thinkabout) are written to `.team/{slug}/plan.md` in the same format as /team IDEATION output.

/team build [slug] detects existing plan.md and proceeds directly to BUILD (HARD-GATE still applies — user must confirm BUILD).

thinkabout writes:
- .team/{slug}/plan.md — FinalPlan
- .team/{slug}/brief.md — FormalBrief
- .team/{slug}/simplicity-scorecard.md — Guard's simplicity assessment
- .team/{slug}/.state — current_phase: "BUILD_READY", mode: "thinkabout"

**thinkabout integration notes:**
- `/team build [slug]` detects existing plan.md and proceeds to BUILD — HARD-GATE still applies, user must confirm
- `current_phase: "BUILD_READY"` means plan exists and is valid; Orchestrator treats this as equivalent to IDEATION having completed with user acceptance
- loop_count is set to 0 initially; /team build increments it normally on success/failure
- If `/team resume [slug]` is called on a thinkabout project in ESCALATED state: re-prompt for BUILD confirmation (HARD-GATE authority preserved)
