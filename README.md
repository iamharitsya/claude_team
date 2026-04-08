# /team — Multi-Agent Orchestration for Claude Code

**Stop hand-holding the AI. Orchestrate it.**

`/team` turns Claude Code into a structured war room. Spin up adversarial agents to debate decisions, stress-test designs, and execute full build pipelines — with a persistent memory that gets smarter every session.

---

## What It Does

### Debate Mode
Tell `/team` to argue something, and it spawns multiple agents who genuinely disagree:

```
/team should we add OAuth2 to our API
/team quickly Redis vs in-memory sessions
/team findbug why is our CI flaky
/team plan add real-time notifications
```

Agents go head-to-head across 2 loops. Contrarians propose falsification tests. Ultrathink mode (auto-triggered for auth, payments, security) exposes full reasoning chains and attacks them. You get a FinalPlan — not just a suggestion.

### Pipeline Mode
For features that need building:

```
/team all add real-time notifications --confirm
```

Runs the full pipeline:
```
IDEATION → BUILD → STRESS → SECURITY → DEPLOY GATE
  ↑         ↓
CORRECTION ←←←←←←← (auto-retry, 2 loops, then escalate)
```

Each phase is a hard gate — it won't proceed without you.

### Memory That Compounds
Every session end runs **MemoryCurator**: extracts what went wrong, what was fixed, what reasoning failed. `.team/learnings.md` grows across sessions. Next time you debate a similar topic, past learnings surface automatically at IDEATION start.

---

## Quick Start

```bash
# Any Claude Code session:
/team quickly Redis or SQLite for session storage
/team plan add GraphQL to our REST API
/team all migrate to server components --confirm
```

---

## Debate Modes

| Command | Mode | Loops | Best For |
|---------|------|-------|----------|
| `/team [x]` | full | 2 | Architectural decisions |
| `/team quickly [x]` | lite | 1 | Binary decisions, fast |
| `/team findbug [x]` | debug | 2 | Root cause investigation |
| `/team plan [x]` | plan | 2 | Feature planning |

**Ultrathink** auto-activates for: `auth`, `security`, `payment`, `OAuth`, `JWT`, `GDPR`, `infrastructure`, `compliance` — and anything involving user safety, financial loss, or data breach. When it triggers, you see: Reasoning Chain → Chain Attack → Steelman → Self-Critique → Synthesis.

---

## Pipeline Phases

```
IDEATION   →  BUILD  →  STRESS  →  SECURITY  →  DEPLOY
  4 agents    4 agents   LoadTest    SAST/DAST    Gate
  (parallel)  (parallel) +EdgeCase  +DepScan     (auto)
               ↓
           CORRECTION (auto-retry)
```

---

## Memory Commands

```bash
/team replay JWT auth        # Surface past learnings for a topic
/team rebuild-patterns       # Regenerate pattern library from all learnings
```

Pre-flight query runs automatically at every IDEATION start — no need to invoke `/team replay` unless you want a explicit refresher before starting something unrelated to the current session.

---

## Install

```
~/.claude/skills/team/
├── SKILL.md
└── references/
    └── state_schema.md
```

Drop into your Claude Code skills directory. Activate with `/team` in any session.

---

## What Gets Built

State lives in `.team/` in your project root:
- `.team/learnings.md` — cross-session learnings database
- `.team/pattern-library.md` — aggregated patterns by category
- `.team/{project}/.state` — phase/loop state (fsynced, not ephemeral)
- `.team/{project}/brief.md` — FormalBrief per session
- `.team/{project}/plan.md` — FinalPlan per session
- `.team/{project}/session-memory.json` — per-session memory capture
- `.team/{project}/correction-log.md` — root cause + fix strategy log
- `.team/decisions.md` — append-only decision log across all projects

---

## Design Principles

1. **Orchestrate, don't micromanage** — wire agents together, don't hand-hold every step
2. **Debate before action** — adversarial testing catches bad reasoning before it becomes code
3. **Phased with hard gates** — each phase gates the next; no auto-advance without you
4. **Memory compounds** — corrections feed the next session's pre-flight query
5. **Zero dead time** — parallel agents, resumable checkpoints

---

## Agent Roster (22 roles)

Debate: `Advocate`, `Skeptic`, `Investigator`, `RootCauseAnalyst`, `FixStrategist`, `Analyst`, `Designer`, `Implementer`, `Contrarian`, `ChainAttacker`, `SteelmanSpecialist`, `SelfCritiqueSpecialist`

Pipeline: `BuilderFrontend`, `BuilderBackend`, `BuilderDevOps`, `BuilderQA`, `LoadTester`, `EdgeCaseAnalyst`, `SecurityAuditor`, `DependencyScanner`, `MemoryCurator`
