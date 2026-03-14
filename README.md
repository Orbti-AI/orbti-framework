<div align="center">

# ORBIT

**Observe, Refine, Build, Integrate, Test** — Structured AI-assisted development for Claude Code.

[![npm version](https://img.shields.io/npm/v/orbit-framework?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/orbit-framework)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/menosjuros/orbit-framework?style=for-the-badge&logo=github&color=181717)](https://github.com/menosjuros/orbit-framework)

<br>

```bash
npx orbit-framework
```

**Works on Mac, Windows, and Linux.**

<br>

*"Quality over speed-for-speed's-sake. In-session context over subagent sprawl."*

<br>

[Why ORBIT](#why-orbit) · [Differentials](#differentials) · [Getting Started](#getting-started) · [The Loop](#the-loop) · [Commands](#commands) · [How It Works](#how-it-works)

</div>

---

## Why ORBIT

Building with Claude Code is incredibly powerful — when you give it the right context.

The problem? **Context rot.** As your session fills up, quality degrades. Subagents spawn with fresh context but return ~70% quality work that needs cleanup. Plans get created but never closed. State drifts. You end up debugging AI output instead of shipping features.

ORBIT is the **scalpel** to the hammer. It is not about going faster — it is about going right the first time, with a system that doesn't forget what it decided, doesn't skip verification, and doesn't leave loose ends.

---

## Differentials

What separates ORBIT from every other AI development framework:

### 1. Structured Methodology — The Loop is Non-Negotiable

Most AI workflows are linear: ask → get code → move on. ORBIT enforces a **closed loop**:

```
PLAN ──▶ APPLY ──▶ UNIFY
  ✓        ✓        ✓     [Loop complete — no orphans]
```

- Every plan **must** close with UNIFY
- UNIFY reconciles what was planned vs. what happened
- No orphan plans. No forgotten decisions. No state drift.

This is the heartbeat of ORBIT. Without it, you have chaos with good intentions.

---

### 2. Context Continuity — The AI That Doesn't Forget

The biggest problem with AI-assisted development is **amnesia between sessions**. ORBIT solves this structurally:

| Problem | ORBIT Solution |
|---------|---------------|
| Session ends, context is lost | `STATE.md` persists everything |
| Next session starts from scratch | `/orbit:resume` restores exact position |
| Decisions evaporate | UNIFY logs all decisions permanently |
| Blockers get forgotten | `STATE.md` tracks blockers explicitly |

When you resume work, ORBIT reads your state and suggests exactly **ONE** next action. No decision fatigue. No "where was I?"

---

### 3. In-Session Context — Quality Over Subagent Sprawl

Other frameworks spawn subagents for everything. ORBIT is deliberate:

| Subagent Cost | Impact |
|---------------|--------|
| Launch overhead | 2,000–3,000 tokens per spawn |
| Fresh context | Starts from scratch every time |
| Resynthesis | Results must be re-integrated |
| Quality gap | ~70% vs. in-session work |
| Rework | Subagent output regularly needs cleanup |

**ORBIT keeps implementation in-session.** Subagents are reserved for what they excel at: discovery and research — tasks where gathering new context IS the job.

---

### 4. Rigorous Verification — Acceptance-Driven Development

Acceptance criteria are not afterthoughts in ORBIT — they are the **foundation**:

- AC defined **before** any task is written
- Every task **references** its AC: `AC-1`, `AC-2`...
- Every task requires a `<verify>` step — no verify, no complete
- BDD format enforced: `Given [precondition] / When [action] / Then [outcome]`

If you can't write the verify step, the task is too vague to execute.

---

### 5. Artisanal Quality — The Scalpel, Not the Hammer

> *"Velocity without structure is just expensive chaos."*

ORBIT prioritizes **craftsmanship**:

- Boundaries are enforced: `DO NOT CHANGE` sections are real constraints, not suggestions
- Deviations are logged: every deviation from the plan is recorded with reason and impact
- Phase transitions are verified: STATE.md, PROJECT.md, and ROADMAP.md must align before moving forward
- Urgent work is tracked separately: decimal phases (2.1, 2.2) for interruptions, integers for planned work

---

### 6. Dynamic Rules via CARL — Lean Context, Always

ORBIT integrates with **[CARL](https://github.com/ChristopherKahler/carl-core)** (Context Augmentation & Reinforcement Layer) — a rule injection system that loads ORBIT's 14 governance rules **only when needed**:

| Without CARL | With CARL |
|---|---|
| Massive static prompts in every session | Rules load on demand |
| Context bloated from the start | Context stays lean |
| Rules apply everywhere, always | Rules activate only in `.orbit/` projects |

---

### ORBIT vs. the Alternatives

| Aspect | Ad-hoc AI | GSD | **ORBIT** |
|--------|-----------|-----|-----------|
| Structure | None | Parallel subagents | Closed loop enforcement |
| Context | Disappears | Fresh per agent | Persisted across sessions |
| Closure | Never | Optional | **Mandatory UNIFY** |
| Criteria | None | Embedded in tasks | **First-class AC section** |
| Quality | Unpredictable | Fast, ~70% | **Crafted, in-session** |
| Rules | None | Static prompts | **CARL dynamic loading** |
| Decisions | Lost | Partial | **Permanently logged** |

See [ORBIT-VS-GSD.md](ORBIT-VS-GSD.md) for the full detailed comparison.

---

## Getting Started

```bash
npx orbit-framework
```

The installer prompts you to choose:
1. **Global** — available in all projects (`~/.claude/`)
2. **Local** — this project only (`./.claude/`)

Verify with `/orbit:help` inside Claude Code.

### Quick Workflow

```bash
# 1. Initialize ORBIT in your project
/orbit:init

# 2. Create a plan for your work
/orbit:plan

# 3. Execute the approved plan
/orbit:apply

# 4. Close the loop (required!)
/orbit:unify

# 5. Check progress anytime
/orbit:progress
```

### Staying Updated

```bash
npx orbit-framework@latest
```

<details>
<summary><strong>Non-interactive Install</strong></summary>

```bash
npx orbit-framework --global   # Install to ~/.claude/
npx orbit-framework --local    # Install to ./.claude/
```

</details>

---

## The Loop

Every unit of work follows this cycle:

```
┌─────────────────────────────────────┐
│  PLAN ──▶ APPLY ──▶ UNIFY          │
│                                     │
│  Define    Execute    Reconcile     │
│  work      tasks      & close       │
└─────────────────────────────────────┘
```

### PLAN

Create an executable plan with:
- **Objective** — What you're building and why
- **Acceptance Criteria** — Given/When/Then definitions of done
- **Tasks** — Specific actions with files, verification, done criteria
- **Boundaries** — What NOT to change

### APPLY

Execute the approved plan:
- Tasks run sequentially
- Each task has verification
- Checkpoints pause for human input when needed
- Deviations are logged

### UNIFY

Close the loop (required!):
- Create SUMMARY.md documenting what was built
- Compare plan vs actual
- Record decisions and deferred issues
- Update STATE.md

**Never skip UNIFY.** Every plan needs closure. This is what separates structured development from chaos.

---

## Commands

ORBIT provides 26 commands organized by purpose. Run `/orbit:help` for the complete reference.

### Core Loop

| Command | What it does |
|---------|--------------|
| `/orbit:init` | Initialize ORBIT in a project |
| `/orbit:plan [phase]` | Create an executable plan |
| `/orbit:apply [path]` | Execute an approved plan |
| `/orbit:unify [path]` | Reconcile and close the loop |
| `/orbit:help` | Show command reference |
| `/orbit:progress [context]` | Smart status + ONE next action |

### Session

| Command | What it does |
|---------|--------------|
| `/orbit:pause [reason]` | Create handoff for session break |
| `/orbit:resume [path]` | Restore context and continue |
| `/orbit:handoff [context]` | Generate comprehensive handoff |

### Roadmap

| Command | What it does |
|---------|--------------|
| `/orbit:add-phase <desc>` | Append phase to roadmap |
| `/orbit:remove-phase <N>` | Remove future phase |

### Milestone

| Command | What it does |
|---------|--------------|
| `/orbit:milestone <name>` | Create new milestone |
| `/orbit:complete-milestone` | Archive and tag milestone |
| `/orbit:discuss-milestone` | Articulate vision before starting |

### Pre-Planning

| Command | What it does |
|---------|--------------|
| `/orbit:discuss <phase>` | Capture decisions before planning |
| `/orbit:assumptions <phase>` | See Claude's intended approach |
| `/orbit:discover <topic>` | Explore options before planning |
| `/orbit:consider-issues` | Triage deferred issues |

### Research

| Command | What it does |
|---------|--------------|
| `/orbit:research <topic>` | Deploy research agents |
| `/orbit:research-phase <N>` | Research unknowns for a phase |

### Specialized

| Command | What it does |
|---------|--------------|
| `/orbit:flows` | Configure skill requirements |
| `/orbit:config` | View/modify ORBIT settings |
| `/orbit:map-codebase` | Generate codebase overview |

### Quality

| Command | What it does |
|---------|--------------|
| `/orbit:verify` | Guide manual acceptance testing |
| `/orbit:plan-fix` | Plan fixes for UAT issues |

---

## How It Works

### Project Structure

```
.orbit/
├── PROJECT.md           # Project context and requirements
├── ROADMAP.md           # Phase breakdown and milestones
├── STATE.md             # Loop position and session state
├── config.md            # Optional integrations
├── SPECIAL-FLOWS.md     # Optional skill requirements
└── phases/
    ├── 01-foundation/
    │   ├── 01-01-PLAN.md
    │   └── 01-01-SUMMARY.md
    └── 02-features/
        ├── 02-01-PLAN.md
        └── 02-01-SUMMARY.md
```

### State Management

**STATE.md** tracks:
- Current phase and plan
- Loop position (PLAN/APPLY/UNIFY markers)
- Session continuity (where you stopped, what's next)
- Accumulated decisions
- Blockers and deferred issues

When you resume work, `/orbit:resume` reads STATE.md and suggests exactly ONE next action. No decision fatigue.

### PLAN.md Structure

```markdown
---
phase: 01-foundation
plan: 01
type: execute
autonomous: true
---

<objective>
Goal, Purpose, Output
</objective>

<context>
@-references to relevant files
</context>

<acceptance_criteria>
## AC-1: Feature Works
Given [precondition]
When [action]
Then [outcome]
</acceptance_criteria>

<tasks>
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/api/auth/login.ts</files>
  <action>Implementation details...</action>
  <verify>curl command returns 200</verify>
  <done>AC-1 satisfied</done>
</task>
</tasks>

<boundaries>
## DO NOT CHANGE
- database/migrations/*
- src/lib/auth.ts
</boundaries>
```

Every task has: files, action, verify, done. If you can't specify all four, the task is too vague.

---

## Troubleshooting

**Commands not found after install?**
- Restart Claude Code to reload slash commands
- Verify files exist in `~/.claude/commands/orbit/` (global) or `./.claude/commands/orbit/` (local)

**Commands not working as expected?**
- Run `/orbit:help` to verify installation
- Re-run `npx orbit-framework` to reinstall

**Loop position seems wrong?**
- Check `.orbit/STATE.md` for current state
- Run `/orbit:progress` for guided next action

**Resuming after a break?**
- Run `/orbit:resume` — it reads state and handoffs automatically

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Claude Code is powerful. ORBIT makes it reliable.**

</div>
