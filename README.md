<div align="center">

# ORBIT

**Observe, Refine, Build, Integrate, Test** — Structured AI-assisted development for Claude Code.

[![npm version](https://img.shields.io/npm/v/orbit-framework?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/orbit-framework)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/menosjuros/orbit-framework?style=for-the-badge&logo=github&color=181717)](https://github.com/menosjuros/orbit-framework)

<br>

```bash
npx github:menosjuros/orbit-framework
```

**Works on Mac, Windows, and Linux.**

<br>

*"Quality over speed-for-speed's-sake. In-session context over subagent sprawl."*

<br>

[What is ORBIT](#what-is-orbit) · [Why ORBIT](#why-orbit) · [Getting Started](#getting-started) · [The Loop](#the-loop) · [Commands](#commands) · [How It Works](#how-it-works)

</div>

---

## What is ORBIT

ORBIT is an acronym — and each letter is a phase of the development cycle:

| Letter | Phase | What happens |
|--------|-------|-------------|
| **O** | **Observe** | Understand the problem before touching code. Map the codebase, gather context, ask the right questions. |
| **R** | **Refine** | Shape the plan with precision. Define acceptance criteria, set boundaries, sharpen the scope. |
| **B** | **Build** | Execute with full in-session focus. Tasks run sequentially, every deviation is logged. |
| **I** | **Integrate** | Reconcile what was planned vs. what was built. Update state, record decisions, close the loop. |
| **T** | **Test** | Verify every task against its acceptance criteria. No verify step, no done. |

This is not a metaphor — these are the actual phases you move through in every unit of work.

---

## Why ORBIT

Building with Claude Code is incredibly powerful — when you give it the right context.

The problem? **Context rot.** As your session fills up, quality degrades. Subagents spawn with fresh context but return ~70% quality work that needs cleanup. Plans get created but never closed. State drifts. You end up debugging AI output instead of shipping features.

ORBIT is the **scalpel**. Not about going faster — about going right the first time, with a system that doesn't forget what it decided, doesn't skip verification, and doesn't leave loose ends.

---

## What Makes ORBIT Different

### The loop is non-negotiable

Every unit of work follows a closed cycle. No plan is left open. No work is done without a definition of done. The UNIFY step — where you reconcile what was planned vs. what actually happened — is mandatory, not optional. This is the heartbeat that keeps everything honest.

### The AI doesn't forget

State persists across sessions. Every decision is logged. Every blocker is tracked. When you come back after a break, ORBIT reads your state and tells you exactly one next action. No ambiguity, no starting over.

### Quality stays in-session

Other workflows spawn subagents for everything and accept the ~70% quality that comes with it. ORBIT keeps implementation work in-session, where context is rich and quality is high. Subagents are reserved for what they do best: discovery and research.

### Acceptance criteria come first

You don't start building until you know what "done" means. Every task references its acceptance criteria. Every task requires a verification step. BDD format — `Given / When / Then` — makes criteria testable, not just readable. If you can't write the verify step, the task is too vague to execute.

### Boundaries are enforced

`DO NOT CHANGE` sections in plans are real constraints. ORBIT enforces them. Phase transitions require state consistency checks. Deviations are documented with reason and downstream impact. Nothing slips through silently.

### Rules load on demand via CARL

ORBIT integrates with **[CARL](https://github.com/ChristopherKahler/carl-core)** — a dynamic rule injection system. Instead of bloating every session with static prompts, ORBIT's 14 governance rules load only when you're inside an `.orbit/` project, and disappear when you're not. Your context stays lean.

---

## Getting Started

### 1. Install ORBIT

Run the installer directly from this repository — no clone needed:

```bash
npx github:menosjuros/orbit-framework
```

The installer prompts you to choose:
1. **Global** — available in all projects (`~/.claude/`)
2. **Local** — this project only (`./.claude/`)

> **Prerequisites:** [Node.js](https://nodejs.org) 18+ and [Claude Code](https://claude.ai/code) installed.

<details>
<summary><strong>Non-interactive install</strong></summary>

```bash
npx github:menosjuros/orbit-framework --global   # Install to ~/.claude/
npx github:menosjuros/orbit-framework --local    # Install to ./.claude/
```

</details>

<details>
<summary><strong>Install via clone</strong></summary>

```bash
git clone https://github.com/menosjuros/orbit-framework.git
cd orbit-framework
node bin/install.js
```

</details>

### 2. Verify the install

Restart Claude Code, then run:

```
/orbit:help
```

You should see the full command reference. If commands are missing, see [Troubleshooting](#troubleshooting).

### 3. Initialize in your project

Open Claude Code inside your project directory and run:

```
/orbit:init
```

This creates the `.orbit/` folder with `PROJECT.md`, `ROADMAP.md`, and `STATE.md` — the core state files ORBIT uses to track your work.

### 4. Run your first loop

```bash
# Define what you're building
/orbit:plan

# Execute the approved plan
/orbit:apply

# Close the loop (required!)
/orbit:unify

# Check progress anytime
/orbit:progress
```

### Common Workflows

**Starting a new project:**
```
/orbit:init → /orbit:plan → /orbit:apply → /orbit:unify
```

**Checking where you are:**
```
/orbit:progress
```

**Resuming work (new session):**
```
/orbit:resume
```

**Pre-planning exploration:**
```
/orbit:discuss 3 → /orbit:assumptions 3 → /orbit:research "topic" → /orbit:plan 3
```

### Key Principles

1. **Loop must complete** — PLAN → APPLY → UNIFY, no shortcuts
2. **State is tracked** — STATE.md knows where you are
3. **Boundaries are real** — Respect `DO NOT CHANGE` sections
4. **Acceptance criteria first** — Define done before starting
5. **Skills are enforced** — Required skills block APPLY until loaded

### Staying Updated

```bash
npx github:menosjuros/orbit-framework
```

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
