<div align="center">

# ORBTI

**Observe, Refine, Build, Integrate, Test** — A structured development loop for Claude Code.

[![npm version](https://img.shields.io/npm/v/orbti-framework?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/orbti-framework)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/menosjuros/orbti-framework?style=for-the-badge&logo=github&color=181717)](https://github.com/menosjuros/orbti-framework)

<br>

```bash
npx github:menosjuros/orbti-framework
```

</div>

---

## What is ORBTI

ORBTI is a framework for Claude Code that keeps AI-assisted development **predictable and reliable** across sessions.

Most AI workflows break down over time: sessions fill up, context degrades, plans get started but never closed, state drifts. ORBTI solves this with a **mandatory closed loop** — every unit of work must be planned, executed, and reconciled before moving on.

### How ORBTI stands out

| | ORBTI | Generic AI workflows |
|--|-------|----------------------|
| **State** | Tracked in `STATE.md` — always knows where you are | Lost between sessions |
| **Planning** | Acceptance criteria before a single line of code | Vague prompts |
| **Execution** | Tasks run sequentially, deviations logged | Freeform, unpredictable |
| **Closure** | INTEGRATE is mandatory — loop can't skip | Plans abandoned mid-way |
| **Research** | Subagents for research only, never implementation | Mixed concerns |
| **Context** | Rich in-session context preserved via pause/resume | Restarts from zero |

---

## Getting Started

**Prerequisites:** [Node.js](https://nodejs.org) 18+ and [Claude Code](https://claude.ai/code).

### Install

```bash
npx github:menosjuros/orbti-framework
```

Choose **global** (`~/.claude/`) to use in all projects, or **local** (`./.claude/`) for this project only.

<details>
<summary>Non-interactive install</summary>

```bash
npx github:menosjuros/orbti-framework --global
npx github:menosjuros/orbti-framework --local
```

</details>

### Initialize

Restart Claude Code, then run:

```
/orbti:init
```

This creates `.orbti/` with `PROJECT.md`, `ROADMAP.md`, and `STATE.md` through a short conversational setup.

Verify with `/orbti:help` — you should see the full command reference.

---

## The Main Loop

The core of ORBTI. Every unit of work follows this cycle — no shortcuts.

```
┌──────────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE          │
│  Define     Execute    Reconcile & close  │
└──────────────────────────────────────────┘
```

```
/orbti:refine     # define what you're building — get plan approved
/orbti:build      # execute the approved plan
/orbti:integrate  # reconcile and close the loop — never skip
```

Repeat for each piece of work.

### REFINE

Creates a `REFINE.md` — the contract for what gets built. Defines:
- **Objective** — what and why
- **Acceptance Criteria** — Given/When/Then definitions of done
- **Tasks** — each with `files`, `action`, `verify`, and `done`
- **Boundaries** — what must NOT change

If you can't fill all four fields per task, the task is too vague to execute.

### BUILD

Executes the approved plan sequentially:
- Each task has a built-in verification step
- Deviations are logged with reason and impact
- Checkpoints pause for human decisions, visual verification, or manual steps

### INTEGRATE

Closes the loop — **never skip this**:
- Compares plan vs what was actually built
- Records decisions and deferred issues
- Updates `STATE.md`
- If it's the last refine in a project: triggers project transition and git commit automatically

---

## Going Deeper

These commands add structure before and around the main loop. Use them for complex projects, unclear scope, or multi-session work.

### Before planning a project

Use these to align before writing a single line of plan:

```
/orbti:cocreate "feature"    # articulate what you want to build
/orbti:observe "topic"       # research technical unknowns before deciding
/orbti:assumptions           # surface Claude's understanding — catch misalignments early
```

**`/orbti:cocreate`** — Conversational exploration of the feature. Helps you articulate goals, scope, and constraints before committing to a plan. Avoids building the wrong thing.

**`/orbti:observe "topic"`** — Deploys research subagents to compare libraries, patterns, or approaches. Produces an `OBSERVE.md` with a recommendation and confidence level. Use when you have a technical unknown that could change the plan.

**`/orbti:assumptions`** — Surfaces what Claude *intends* to do before planning. Run this after cocreate and observe to catch misalignments early, not mid-build.

---

### The full loop

```
/orbti:refine      # plan informed by cocreate + observe
/orbti:build       # execute
/orbti:test        # verify against acceptance criteria
/orbti:integrate   # close the loop
```

**`/orbti:test`** — Auto-detects the project's test runner, writes missing tests, and maps results to each AC. Falls back to guided manual UAT if no runner is found.

Flags:
- `--manual` — skip auto-detection, go straight to manual UAT

---

### Configuration

All optional features are **disabled by default**. Enable via:

```
/orbti:config
```

Available integrations:

| Integration | What it does | Default |
|-------------|-------------|---------|
| **Agent Teams** | Parallel research on `/orbti:observe`, code review on `/orbti:integrate` | OFF |
| **Test Writer** | Writes integration tests during `/orbti:build`, one per AC | OFF |
| **E2E (Playwright CLI)** | Browser-based tests via external Playwright CLI | OFF |

When **Test Writer** is enabled:
- Agent Teams also ON → builder + test-writer run simultaneously
- Agent Teams OFF → test written sequentially after each task

#### Enable Agent Teams

```
/orbti:config → Agent Teams → Enable
```

Also writes `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` to `.claude/settings.json` automatically. Restart Claude Code after enabling.

#### Enable E2E (Playwright CLI)

Playwright CLI is an external tool — install it independently first:

```bash
npm install -g @playwright/cli@latest
playwright-cli install --skills
playwright-cli install chromium
```

Then enable in ORBTI:

```
/orbti:config → Playwright CLI E2E → Enable
```

ORBTI will verify the binary and skill are installed before enabling.

---

### Session management

Work across multiple sessions and multiple projects without losing context:

```
/orbti:pause         # pause active project (if only one active)
/orbti:pause auth    # pause specific project by name or number
/orbti:resume        # restore context — shows all projects, suggests next action
```

**`/orbti:pause [project]`** — Marks the project as `⏸ Paused` and creates a handoff at `.orbti/projects/{project}/HANDOFF-{date}.md`. If multiple projects are active and no argument is given, asks which to pause.

**`/orbti:resume`** — Shows the full Projects Overview with every project's status and loop position. If multiple projects are active or paused, lets you choose which to continue. Then gives exactly one next action.

```
Projects Overview:
│  01  auth-service    1/3   ⏸ Paused        ✓ ◉ ○  │
│  02  dashboard       0/2   🔵 In Progress  ◉ ○ ○  │
│  03  api-layer       0/2   ○ Pending       ○ ○ ○  │
```

---

### Milestones

Group related projects into milestones for larger initiatives:

```
/orbti:cocreate-milestone    # align on milestone vision before starting
/orbti:milestone "name"      # create milestone and add projects to ROADMAP.md
...                          # work through projects with the main loop
/orbti:complete-milestone    # archive, tag, and close the milestone
```

**`/orbti:cocreate-milestone`** — Like cocreate, but for the full milestone. Define the vision, success criteria, and scope before creating any projects.

**`/orbti:complete-milestone`** — Archives completed milestone to `.orbti/milestones/`, creates a git tag, and prepares `STATE.md` for the next milestone.

---

## Command Reference

### Core loop

| Command | Description |
|---------|-------------|
| `/orbti:refine` | Plan the next unit of work — creates `REFINE.md` with objective, ACs, tasks, and boundaries |
| `/orbti:build` | Execute the approved plan sequentially, with verification at each step |
| `/orbti:build-bg` | Run the approved plan as a background agent — unattended (requires `autonomous: true`) |
| `/orbti:integrate` | Reconcile plan vs actual, update state, close the loop — never skip |
| `/orbti:test` | Verify against acceptance criteria — auto-detects test runner, falls back to manual UAT |

### Before planning

| Command | Description |
|---------|-------------|
| `/orbti:cocreate "feature"` | Conversational exploration — articulate goals and scope before planning |
| `/orbti:observe "topic"` | Deploy research subagents to compare options — produces `OBSERVE.md` |
| `/orbti:assumptions` | Surface what Claude intends to do — catch misalignments before build |

### Session management

| Command | Description |
|---------|-------------|
| `/orbti:pause [project]` | Pause a project — argument required when multiple projects are active |
| `/orbti:resume` | Show all projects overview, pick which to continue, get one next action |
| `/orbti:progress` | Smart status — shows where you are and suggests the next step |

### Project setup

| Command | Description |
|---------|-------------|
| `/orbti:init` | Initialize ORBTI in a project — creates `.orbti/` with PROJECT.md, ROADMAP.md, STATE.md |
| `/orbti:config` | Enable/disable integrations (Agent Teams, Test Writer, E2E) |

### Milestones

| Command | Description |
|---------|-------------|
| `/orbti:cocreate-milestone` | Define milestone vision and success criteria before creating projects |
| `/orbti:milestone "name"` | Create a milestone and add projects to ROADMAP.md |
| `/orbti:complete-milestone` | Archive milestone, create git tag, prepare for next milestone |

### Utilities

| Command | Description |
|---------|-------------|
| `/orbti:help` | Show all available commands |
| `/orbti:add-phase` | Add a new project to the current milestone |
| `/orbti:remove-phase` | Remove a future (not started) project |
| `/orbti:refine-fix` | Plan fixes for issues found during `/orbti:test` |

---

## How It Works

### File structure

```
.orbti/
├── PROJECT.md           # Project context and goals
├── ROADMAP.md           # Milestones and project breakdown
├── STATE.md             # Global dashboard — all projects overview + current focus
├── config.md            # Optional integrations
├── SPECIAL-FLOWS.md     # Optional skill requirements per project
├── milestones/          # Archived completed milestones
│   └── v1.0-launch.md
└── projects/            # Flat numbering — never restarts at 01
    ├── 01-auth/
    │   ├── 01-01-REFINE.md
    │   ├── 01-01-INTEGRATE.md
    │   └── HANDOFF-2026-03-17.md   # Created on /orbti:pause, deleted on complete
    └── 02-dashboard/
        ├── 02-01-REFINE.md
        └── 02-01-INTEGRATE.md
```

Projects are numbered continuously across milestones. Milestone grouping lives in `ROADMAP.md`, not in the folder structure.

### State management

A single `STATE.md` acts as the global dashboard — it has two layers:

- **Projects Overview** — one row per project with loop count, status (`○ Pending / ⏸ Paused / 🔵 In Progress / ✅ Complete`), and current loop position
- **Current Focus** — deep view of the active project with loop position and last activity

`/orbti:resume` reads it and either gives ONE next action (single active project) or shows the Overview and lets you pick which project to continue.

### REFINE.md structure

```markdown
---
project: 01-auth
refine: 01
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
## AC-1: Login works
Given a valid user
When they submit credentials
Then they receive a JWT token
</acceptance_criteria>

<tasks>
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/api/auth/login.ts</files>
  <action>Implementation details...</action>
  <verify>curl returns 200 with token</verify>
  <done>AC-1 satisfied</done>
</task>
</tasks>

<boundaries>
## DO NOT CHANGE
- database/migrations/*
- src/lib/auth.ts
</boundaries>
```

### Task types

| Type | Use for |
|------|---------|
| `auto` | Fully autonomous execution |
| `checkpoint:decision` | Choices requiring human input |
| `checkpoint:human-verify` | Visual or functional verification |
| `checkpoint:human-action` | Manual steps (deploy, external service) |

---

## Troubleshooting

**Commands not found after install?**
Restart Claude Code. Verify files exist in `~/.claude/commands/orbti/` (global) or `./.claude/commands/orbti/` (local).

**Commands not working as expected?**
Run `/orbti:help` to verify installation. Re-run `npx github:menosjuros/orbti-framework` to reinstall.

**Loop position seems wrong?**
Check `.orbti/STATE.md` directly. Run `/orbti:progress` for guided next action.

**Resuming after a break?**
Run `/orbti:resume` — reads STATE.md and handoffs automatically.

---

## License

MIT. See [LICENSE](LICENSE).

This project is a fork of [CARL](https://github.com/ChristopherKahler/carl) by Chris Kahler.

---

<div align="center">

**Claude Code is powerful. ORBTI makes it reliable.**

</div>
