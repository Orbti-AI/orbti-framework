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

*"Quality over speed-for-speed's-sake. In-session context over subagent sprawl."*

</div>

---

## What is ORBIT

ORBIT is a structured development loop for Claude Code. Each letter is a phase:

| | Phase | What happens |
|--|-------|-------------|
| **O** | **Observe** | Optional pre-planning step. Research technical options, compare libraries, and make architecture decisions before planning. Produces an OBSERVE.md with recommendation and confidence level. |
| **R** | **Refine** | Shape the plan. Acceptance criteria, scope, boundaries. |
| **B** | **Build** | Execute in-session. Sequential tasks, every deviation logged. |
| **I** | **Integrate** | Reconcile plan vs reality. Record decisions, close the loop. |
| **T** | **Test** | Verify against acceptance criteria. No verify, no done. |

The problem ORBIT solves: **context rot**. As sessions fill up, quality degrades. Plans get created but never closed. State drifts. ORBIT keeps implementation in-session (where context is rich), reserves subagents for research, and enforces a closed loop so nothing slips through.

---

## Getting Started

### 1. Install

```bash
npx github:menosjuros/orbit-framework
```

> **Prerequisites:** [Node.js](https://nodejs.org) 18+ and [Claude Code](https://claude.ai/code) installed.

The installer prompts you to choose:
- **Global** — available in all projects (`~/.claude/`)
- **Local** — this project only (`./.claude/`)

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

### 3. Initialize a project

```
/orbit:init
```

Creates `.orbit/` with `PROJECT.md`, `ROADMAP.md`, and `STATE.md` through a short conversational setup.

### 4. Run your first loop

```bash
/orbit:refine     # Define what you're building
/orbit:build      # Execute the approved plan
/orbit:integrate  # Close the loop (required)
```

### Staying updated

```bash
npx github:menosjuros/orbit-framework
```

---

## Workflows

### Minimal — day-to-day

The core loop. Use this every day for any unit of work.

```
/orbit:init          # once per project

/orbit:refine        # define the plan, get approval
/orbit:build         # execute
/orbit:integrate     # close the loop — never skip
```

Repeat `refine → build → integrate` for each piece of work.

---

### Full — maximum structure and clarity

Combine all tools for complex projects with unclear scope, technical unknowns, and team handoffs.

```
/orbit:init

/orbit:cocreate-milestone    # align on milestone vision
/orbit:milestone             # create projects in ROADMAP.md

# Before planning a project
/orbit:cocreate 3            # articulate what you want to build
/orbit:assumptions 3         # surface Claude's understanding — catch misalignments early
/orbit:observe "topic"       # resolve technical unknowns

# The loop
/orbit:refine 3              # plan informed by cocreate + observe
/orbit:build
/orbit:test                  # verify against acceptance criteria
/orbit:integrate             # close the loop

# Session management
/orbit:pause                 # safe stop with full context preserved
/orbit:resume                # restore and continue in next session

/orbit:complete-milestone
```

---

## Common Workflows

**Resuming after a break (new session):**
```
/orbit:resume
```

**Checking where you are:**
```
/orbit:progress
```

**Pausing mid-session:**
```
/orbit:pause → (new session) → /orbit:resume
```

**Research before planning:**
```
/orbit:research "JWT best practices" → /orbit:refine 2
```

**Triage deferred issues:**
```
/orbit:consider-issues → /orbit:refine
```

---

## The Loop

```
┌──────────────────────────────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE                              │
│  Define    Execute    Reconcile & close                      │
└──────────────────────────────────────────────────────────────┘
```

### REFINE

Create a LOOP.md with:
- **Objective** — What you're building and why
- **Acceptance Criteria** — Given/When/Then definitions of done
- **Tasks** — Files, action, verify, done for each step
- **Boundaries** — What NOT to change

### BUILD

Execute the approved plan:
- Tasks run sequentially
- Each task has a verification step
- Checkpoints pause for human input (decision, verify, action)
- Deviations are logged with reason and impact

### INTEGRATE

Close the loop — **never skip this**:
- Create INTEGRATE.md documenting what was built
- Compare plan vs actual
- Record decisions and deferred issues
- Update STATE.md
- If last plan in phase: triggers phase transition and git commit automatically

---

## Key Principles

1. **Loop must complete** — REFINE → BUILD → INTEGRATE, no shortcuts
2. **State is tracked** — STATE.md always knows where you are
3. **Boundaries are real** — `DO NOT CHANGE` sections are enforced
4. **Acceptance criteria first** — define done before building
5. **Skills are enforced** — required skills block BUILD until loaded
6. **Quality stays in-session** — subagents for research only, not implementation

---

## Commands

Run `/orbit:help` for the full reference.

### Core Loop

| Command | What it does |
|---------|--------------|
| `/orbit:init` | Initialize ORBIT in a project |
| `/orbit:refine [phase]` | Create an executable plan |
| `/orbit:build [path]` | Execute an approved plan |
| `/orbit:integrate [path]` | Reconcile and close the loop |
| `/orbit:progress [context]` | Smart status + ONE next action |
| `/orbit:help` | Show command reference |

### Session

| Command | What it does |
|---------|--------------|
| `/orbit:pause [reason]` | Create handoff for session break |
| `/orbit:resume [path]` | Restore context and continue |
| `/orbit:handoff [context]` | Generate comprehensive handoff |

### Pre-Planning

| Command | What it does |
|---------|--------------|
| `/orbit:observe <topic>` | Research technical options and make decisions before planning |
| `/orbit:cocreate <project>` | Articulate project vision and goals before planning |
| `/orbit:assumptions <phase>` | Surface Claude's intended approach |
| `/orbit:consider-issues` | Triage deferred issues |

### Research

| Command | What it does |
|---------|--------------|
| `/orbit:research <topic>` | Deploy research agents |
| `/orbit:research-phase <N>` | Research unknowns for a project |

### Roadmap

| Command | What it does |
|---------|--------------|
| `/orbit:add-project <desc>` | Append project to roadmap |
| `/orbit:remove-project <N>` | Remove future project |

### Milestones

| Command | What it does |
|---------|--------------|
| `/orbit:milestone <name>` | Create new milestone |
| `/orbit:complete-milestone` | Archive and tag milestone |
| `/orbit:cocreate-milestone` | Articulate vision before starting a milestone |

### Specialized

| Command | What it does |
|---------|--------------|
| `/orbit:map-codebase` | Generate codebase overview |
| `/orbit:flows` | Configure skill requirements |
| `/orbit:config` | View/modify ORBIT settings |

### Quality

| Command | What it does |
|---------|--------------|
| `/orbit:test` | Guide manual acceptance testing |
| `/orbit:plan-fix` | Plan fixes for UAT issues |

---

## How It Works

### Project structure

```
.orbit/
├── PROJECT.md           # Project context and goals
├── ROADMAP.md           # Phase breakdown and milestones
├── STATE.md             # Loop position and session state
├── config.md            # Optional integrations
├── SPECIAL-FLOWS.md     # Optional skill requirements
├── milestones/          # Archived completed milestones
│   └── v0.1-core-loop.md
└── projects/            # All projects — flat numbering across milestones
    ├── 01-foundation/
    │   ├── 01-01-LOOP.md
    │   └── 01-01-INTEGRATE.md
    └── 02-features/
        ├── 02-01-LOOP.md
        └── 02-01-INTEGRATE.md
```

Projects are numbered continuously across milestones — they never restart at 01. Milestone grouping lives in `ROADMAP.md`, not in the folder structure.

### State management

**STATE.md** tracks current project, loop position (REFINE/BUILD/INTEGRATE), session continuity, accumulated decisions, and blockers. `/orbit:resume` reads it and gives exactly ONE next action — no decision fatigue.

### LOOP.md structure

```markdown
---
project: 01-foundation
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

Every task requires `files`, `action`, `verify`, `done`. If you can't fill all four, the task is too vague to execute.

### Task types

| Type | Use for |
|------|---------|
| `auto` | Fully autonomous execution |
| `checkpoint:decision` | Choices requiring human input |
| `checkpoint:human-verify` | Visual/functional verification |
| `checkpoint:human-action` | Manual steps (rare) |

### CARL integration

ORBIT integrates with **[CARL](https://github.com/ChristopherKahler/carl-core)** — a dynamic rule injection system. ORBIT's governance rules load only when inside an `.orbit/` project and disappear when you're not. Context stays lean.

---

## Troubleshooting

**Commands not found after install?**
- Restart Claude Code to reload slash commands
- Verify files exist in `~/.claude/commands/orbit/` (global) or `./.claude/commands/orbit/` (local)

**Commands not working as expected?**
- Run `/orbit:help` to verify installation
- Re-run `npx github:menosjuros/orbit-framework` to reinstall

**Loop position seems wrong?**
- Check `.orbit/STATE.md` directly
- Run `/orbit:progress` for guided next action

**Resuming after a break?**
- Run `/orbit:resume` — reads STATE.md and handoffs automatically

---

## License

MIT. See [LICENSE](LICENSE).

---

<div align="center">

**Claude Code is powerful. ORBIT makes it reliable.**

</div>
