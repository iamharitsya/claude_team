# /team — Multi-Agent Orchestration for Claude Code

<p align="center">
  <a href="https://github.com/iamharitsya/claude_team"><img src="https://img.shields.io/github/stars/iamharitsya/claude_team?style=for-the-badge&logo=github&color=D97757&labelColor=1e1e2e" alt="Stars"/></a>
  <a href="https://github.com/iamharitsya/claude_team/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-22c55e?style=for-the-badge&labelColor=1e1e2e" alt="License"/></a>
</p>

<h3 align="center">Stop hand-holding the AI. Orchestrate it.</h3>

<p align="center">
  Multi-agent debate + build pipeline for Claude Code. 22 agent roles, adversarial reasoning,<br/>
  phased execution with human checkpoints, and persistent cross-session memory.<br/>
  <b>4 debate modes</b> &bull; <b>Full pipeline</b> &bull; <b>22 agents</b> &bull; <b>Persistent memory</b>
</p>

---

## The Problem

You ask Claude to "plan our auth system" and get a wall of text. You ask it to "build it" and it writes code that doesn't compile. You spend half the session course-correcting instead of building.

The root cause: single-agent AI can't stress-test its own reasoning. It writes the code, then defends it. You catch the bugs anyway.

## The Solution

`/team` runs your problem through adversarial agents. A Contrarian attacks the reasoning. A ChainAttacker exposes logical flaws. Ultrathink mode (auto-triggered for auth, payments, security) runs 5 phases of reasoning stress-tests before a single line of code is written.

The debate produces a FinalPlan — not a suggestion. The pipeline builds it: IDEATION → BUILD → STRESS → SECURITY → DEPLOY GATE. MemoryCurator runs at session end and feeds everything that went wrong into `.team/learnings.md`. The next session starts smarter.

```
/team plan add OAuth2 authentication
  → Analyst: "Here's the quantified risk surface..."
  → Designer: "Option A (OAuth2 PKCE) vs Option B (SAML)..."
  → Implementer: "Timeline: 3 weeks. Blockers: ..."
  → Contrarian: "Failure mode: token refresh race. Falsification test: ..."
  → Synthesis: FinalPlan with confidence score

/team all add OAuth2 authentication --confirm
  → IDEATION (full debate, FinalPlan produced)
  → BUILD (4 parallel builders: Frontend + Backend + DevOps + QA)
  → STRESS (LoadTester + EdgeCaseAnalyst)
  → SECURITY (SecurityAuditor + DependencyScanner)
  → DEPLOY GATE (auto: APPROVED / REJECTED / ESCALATED)
```

---

## Install (30 seconds)

```bash
# Copy into your Claude Code skills directory
cp -r skills/team ~/.claude/skills/team/

# Activate in any session
/team quickly Redis or in-memory sessions
/team plan add real-time notifications
/team all migrate to server components --confirm
```

For detailed installation and setup, see [docs/](#) (coming soon).

---

## What's New

- **MemoryCurator** — Auto-runs at session end: extracts `[LEARN]` blocks from agent outputs, deduplicates, writes to `.team/learnings.md`
- **Pre-flight query** — Every IDEATION start surfaces top 5 relevant past learnings automatically
- **`/team replay [topic]`** — On-demand surfacing of past learnings for any topic
- **`/team rebuild-patterns`** — Regenerate `.team/pattern-library.md` from all accumulated learnings
- **Correction log** — Root cause + fix strategy captured in `.team/{slug}/correction-log.md`
- **Ultrathink auto-trigger** — Activates on: `auth`, `security`, `payment`, `OAuth`, `JWT`, `GDPR`, `infrastructure`, `compliance` — and any topic involving user safety, financial loss, or data breach

---

## How /team Compares

| Feature | /team | Single-agent chat | Built-in `/team` |
|---------|------:|:-----------------:|:----------------:|
| Adversarial debate (4+ agents) | **Yes** | No | Some |
| Contrarian with falsification tests | **Yes** | No | No |
| Ultrathink (5-phase reasoning chain attack) | **Yes** | No | No |
| Full BUILD → STRESS → SECURITY pipeline | **Yes** | No | No |
| HARD-GATE between IDEATION and BUILD | **Yes** | N/A | No |
| Relay lock (main agent can't contaminate) | **Yes** | No | No |
| Cross-session persistent memory | **Yes** | No | No |
| Pre-flight query at IDEATION start | **Yes** | No | No |
| CORRECTION/ESCALATED state machine | **Yes** | No | No |
| Atomic fsync writes with rollback | **Yes** | No | No |
| Agent roles | 22 | 1 | 5-8 |

---

## Try It

```bash
/team quickly Redis or in-memory sessions          # Quick binary decision (1 loop, 2 agents)
/team should we add OAuth2 to our API              # Full debate (2 loops, 4 agents)
/team findbug why is our CI flaky                   # Debug mode (root cause investigation)
/team plan add real-time notifications              # Plan mode → FinalPlan
/team all add notifications --confirm              # Full pipeline with checkpoints
/team replay JWT auth                               # Surface past learnings
/team rebuild-patterns                              # Regenerate pattern library
```

---

## What's Inside

### 4 Debate Modes

| Mode | Agents | Loops | When to Use |
|------|--------|-------|-------------|
| **full** | Analyst, Designer, Implementer, Contrarian | 2 | Complex architectural decisions |
| **lite** | Advocate, Skeptic | 1 | Fast binary decisions |
| **debug** | Investigator, RootCauseAnalyst, FixStrategist, Contrarian | 2 | Root cause investigation |
| **plan** | Analyst, Designer, Implementer, Contrarian | 2 | Feature planning and build recommendations |

**Ultrathink** auto-activates for high-stakes topics. Runs 5 phases instead of 4:
`Reasoning Chain → Chain Attack → Steelman → Self-Critique + Revision → Synthesis`

### Full Pipeline (5 Phases)

```
IDEATION   →  BUILD  →  STRESS  →  SECURITY  →  DEPLOY GATE
  4 agents    4 agents   LoadTest    SAST/DAST    (auto-decision)
  (parallel)  (parallel) +EdgeCase   +DepScan
               ↓
           CORRECTION (auto-retry on failure)
```

### 22 Agent Roles

| Role | Purpose |
|------|---------|
| **Advocate** | Argue FOR the proposed approach |
| **Skeptic** | Argue AGAINST or challenge |
| **Investigator** | Gather evidence, map symptoms, rank hypotheses |
| **RootCauseAnalyst** | Trace failure chain: symptom → intermediate → root cause |
| **FixStrategist** | Propose 2-3 fix options with effort/risk tradeoffs |
| **Analyst** | Evidence, data, quantified assessments |
| **Designer** | UX and architecture options with tradeoffs |
| **Implementer** | Feasibility, cost, timeline, ownership |
| **Contrarian** | Devil's advocate, failure modes + falsification tests |
| **ChainAttacker** | *(Ultrathink)* Attack reasoning chains, not conclusions |
| **SteelmanSpecialist** | *(Ultrathink)* Argue strongest version of opposition |
| **SelfCritiqueSpecialist** | *(Ultrathink)* Find own weakest point, genuinely revise |
| **BuilderFrontend** | Implement frontend deliverables |
| **BuilderBackend** | Implement backend deliverables |
| **BuilderDevOps** | Infrastructure, Docker, CI/CD |
| **BuilderQA** | Tests, fixtures, test results |
| **LoadTester** | p99 latency, error rate, throughput |
| **EdgeCaseAnalyst** | Critical edge case identification and testing |
| **SecurityAuditor** | SAST + DAST security scans |
| **DependencyScanner** | CVE audit on dependencies |
| **MemoryCurator** | *(Session end)* Extract learnings, deduplicate, update `.team/learnings.md` |

### 8 Commands

| Command | What It Does |
|---------|-------------|
| `/team [x]` | Full debate — 2 loops, 4 agents |
| `/team quickly [x]` | Lite debate — 1 loop, 2 agents |
| `/team findbug [x]` | Debug — 4 agents, root cause investigation |
| `/team plan [x]` | Plan — produce FinalPlan |
| `/team all [x]` | Full pipeline with human checkpoints |
| `/team replay [topic]` | Surface relevant past learnings for a topic |
| `/team rebuild-patterns` | Regenerate pattern library from learnings |
| `/team stop` | Abort active session, release relay lock |

---

## How It Works

### State Machine

```
IDLE → IDEATION → BUILD → STRESS → SECURITY → DEPLOY
              ↑         ↓
           CORRECTION ←←←←←←← (auto-retry, 2 loops max)
```

Every phase allows 2 loops before escalation. The HARD-GATE between IDEATION and BUILD is non-negotiable — `/team` will not proceed to BUILD without your explicit confirmation.

### Relay Lock

Once `/team` is invoked, the main Claude Code agent becomes a relay. It forwards all messages to the Orchestrator sub-agent and **cannot** spawn agents, write files, or run commands independently. This prevents the parent session from contaminating the debate or execution.

### Memory Flow

```
Session end (DEPLOY→IDLE or ESCALATED→IDLE)
  → MemoryCurator runs (before relay lock releases)
  → Reads all .agent_outputs/
  → Extracts [LEARN] blocks or infers learnings from agent roles
  → Deduplicates against .team/learnings.md
  → Finalizes session-memory.json with ultrathink aggregation

Next session IDEATION start
  → Pre-flight query reads .team/learnings.md
  → Surfaces top 5 learnings matching topic_tags
  → Escalation warnings for previously-failed categories
```

---

## Memory Files

| File | Purpose | Lifetime |
|------|---------|----------|
| `.team/learnings.md` | Cross-session append-only learnings | Permanent |
| `.team/pattern-library.md` | Aggregated patterns by category | Permanent |
| `.team/decisions.md` | Append-only decisions log | Permanent |
| `.team/index.md` | Project registry | Permanent |
| `.team/{slug}/session-memory.json` | Per-session: ultrathink stats, topic tags, outcomes | Permanent |
| `.team/{slug}/correction-log.md` | Root cause + fix strategy per CORRECTION | Permanent |
| `.team/{slug}/.state` | Phase/loop state (fsynced) | Permanent |

---

## Core Patterns

| Pattern | What It Does |
|---------|-------------|
| **Adversarial Debate** | Contrarian + ChainAttacker stress-test reasoning before code is written |
| **Phased Execution** | Each phase gates the next; no auto-advance without you |
| **Relay Lock** | Main agent becomes relay; cannot contaminate debate or execution |
| **Self-Correcting Memory** | MemoryCurator extracts learnings; pre-flight query surfaces them |
| **Correction Loop** | Phase fails → CORRECTION enters → root cause captured → fixed → retry |
| **Ultrathink** | 5-phase reasoning exposure: chain attack + steelman + self-critique |
| **Parallel Builders** | Frontend + Backend + DevOps + QA run simultaneously in BUILD |
| **HARD-GATE** | IDEATION → BUILD requires explicit confirmation; never auto-proceeds |

---

## Architecture

```
You: /team all add notifications
     │
     ▼
Main Agent ──(relay lock)──→ Orchestrator
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
   IDEATION              BUILD (parallel)         DEPLOY
   4 agents              Frontend                Gate
   (Position →           Backend                 │
   Contrarian →          DevOps               ┌──┴──┐
   Rebuttal →            QA                   │pass │
   Synthesis)                                  │fail │
                              │             └──┬──┘
        ┌─────────────────────┼─────────────────────┘
        ▼                     ▼
   STRESS                 SECURITY
   LoadTester +           SecurityAuditor +
   EdgeCaseAnalyst       DependencyScanner
        │                     │
        └──────────┬──────────┘
                   ▼
              MemoryCurator
        (extracts learnings → .team/learnings.md)
                   │
                   ▼
            Relay lock releases
```

---

## Philosophy

1. **Orchestrate, don't micromanage** — Wire agents together; don't hand-hold every step
2. **Debate before action** — Adversarial testing catches bad reasoning before it becomes code
3. **Phased with hard gates** — Each phase gates the next; IDEATION won't auto-advance to BUILD
4. **Memory compounds** — Every correction feeds the next session's pre-flight query
5. **Trust but verify** — Let agents work; review at checkpoints

---

## Known Limitations

- **Skill is a YAML prompt specification** — It defines orchestrator logic, agent prompts, and state machines. It runs on Claude Code's built-in agent infrastructure. No binary or server.
- **BUILD agents are LLM prompts** — They produce code that works if the LLM can write and verify files in your environment.
- **STRESS/SECURITY agents need infrastructure** — `LoadTester`, `EdgeCaseAnalyst`, `SecurityAuditor`, `DependencyScanner` require running servers or test tooling. They'll no-op without it.
- **Pre-flight memory starts empty** — Value compounds as `.team/learnings.md` grows over sessions.
- **Relay lock is advisory in single-user environments** — A determined process could delete the lock file. For multi-user environments, use a Redis lock per `references/state_schema.md`.

---

## Related Projects

| [superpowers](https://github.com/obra/superpowers) | Claude Code built-in `/team` |
| [thinkabout](https://github.com/thinkabout-team/thinkabout) | Intent clarification before `/team` execution |

---

<p align="center">
  <br/>
  <b>If /team saves you time, star the repo so others can find it.</b>
  <br/><br/>
  <a href="https://github.com/iamharitsya/claude_team/stargazers"><img src="https://img.shields.io/github/stars/iamharitsya/claude_team?style=for-the-badge&logo=github&color=D97757&labelColor=1e1e2e" alt="Stars"/></a>
  <br/><br/>
  <a href="https://github.com/iamharitsya/claude_team">GitHub</a> &bull;
  <a href="https://github.com/iamharitsya/claude_team/issues">Report Issues</a>
</p>
