---
name: orbti:help
description: Show available ORBTI commands and usage guide
---

<model>haiku</model>

<objective>
Display the complete ORBTI command reference.

Output ONLY the reference content below. Do NOT add:

- Project-specific analysis
- Git status or file context
- Next-step suggestions
- Any commentary beyond the reference
</objective>

<reference>
# ORBTI Command Reference

**ORBTI** (Cocreate, Refine, Build, Integrate, Test) is a structured AI-assisted development framework for Claude Code.

## The Loop

Every unit of work follows this cycle:

```
┌─────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE          │
│                                     │
│  Define    Execute    Reconcile     │
│  work      tasks      & close       │
└─────────────────────────────────────┘
```

**Never skip INTEGRATE.** Every refine needs a summary.

## Quick Start

1. `/orbti:init` - Initialize ORBTI in your project
2. `/orbti:refine` - Create a refine for your work
3. `/orbti:build` - Execute the approved refine
4. `/orbti:integrate` - Close the loop with summary

## Commands Overview

| Category | Commands |
|----------|----------|
| Core Loop | refine, build, integrate, test, help, status |
| Session | pause, resume, progress, handoff |
| Roadmap | add-project, remove-project |
| Milestone | milestone, complete-milestone, observe-milestone |
| Pre-Planning | observe, assumptions, cocreate, consider-issues |
| Research | research, research-phase |
| Specialized | skills, config, map-codebase |
| Quality | test, refine-fix |

---

## Core Loop Commands

### `/orbti:init`
Initialize ORBTI in a project.

- Creates `.orbti/` directory structure
- Creates PROJECT.md, STATE.md, ROADMAP.md
- Prompts for project context and phases
- Optionally configures integrations (SonarQube, etc.)

Usage: `/orbti:init`

---

### `/orbti:refine [project]`
Enter REFINE phase - create an executable refine.

- Reads current state from STATE.md
- Creates REFINE.md with tasks, acceptance criteria, boundaries
- Populates skills section from SKILLS.md (if configured)
- Updates loop position

Usage: `/orbti:refine` (auto-detects next project)
Usage: `/orbti:refine 3` (specific project)

---

### `/orbti:build [refine-path]`
Execute an approved REFINE.md file.

- **Blocks if required skills not loaded** (from SKILLS.md)
- Validates refine exists and hasn't been executed
- Executes tasks sequentially
- Handles checkpoints (decision, human-verify, human-action)
- Reports completion and prompts for INTEGRATE

Usage: `/orbti:build`
Usage: `/orbti:build .orbti/projects/01-foundation/01-01-REFINE.md`

---

### `/orbti:integrate [refine-path]`
Reconcile refine vs actual and close the loop.

- Creates INTEGRATE.md documenting what was built
- Audits skill invocations (if SPECIAL-FLOWS.md configured)
- Records decisions made, deferred issues
- Updates STATE.md with loop closure
- **Required** - never skip this step

Usage: `/orbti:integrate`
Usage: `/orbti:integrate .orbti/projects/01-foundation/01-01-REFINE.md`

---

### `/orbti:help`
Show this command reference.

Usage: `/orbti:help`

---

### `/orbti:status` *(deprecated)*
> Use `/orbti:progress` instead.

Shows current loop position. Deprecated in favor of `/orbti:progress` which provides better routing.

---

## Session Commands

### `/orbti:pause [reason]`
Create handoff file and prepare for session break.

- Creates HANDOFF.md with complete context
- Updates STATE.md session continuity section
- Designed for context limits or multi-session work

Usage: `/orbti:pause`
Usage: `/orbti:pause "switching to other project"`

---

### `/orbti:resume [handoff-path]`
Restore context from handoff and continue work.

- Reads STATE.md and any HANDOFF files
- Determines current loop position
- Suggests exactly ONE next action
- Archives consumed handoffs

Usage: `/orbti:resume`
Usage: `/orbti:resume .orbti/HANDOFF-context.md`

---

### `/orbti:progress [context]`
Smart status with routing - suggests ONE next action.

- Shows milestone and phase progress visually
- Displays current loop position
- Suggests exactly ONE next action (prevents decision fatigue)
- Accepts optional context to tailor suggestion
- Warns about context limits

Usage: `/orbti:progress`
Usage: `/orbti:progress "I only have 30 minutes"`

---

### `/orbti:handoff [context]`
Generate comprehensive session handoff document.

- Creates detailed handoff for complex session breaks
- Captures decisions, progress, blockers, next steps
- More thorough than `/orbti:pause`

Usage: `/orbti:handoff`
Usage: `/orbti:handoff "phase10-audit"`

---

## Roadmap Commands

### `/orbti:add-project <description>`
Append a new project to the roadmap.

- Adds project to end of ROADMAP.md
- Updates project numbering
- Records in STATE.md decisions

Usage: `/orbti:add-project "API Authentication Layer"`

---

### `/orbti:remove-project <number>`
Remove a future (not started) project from roadmap.

- Cannot remove completed or in-progress projects
- Renumbers subsequent projects
- Updates ROADMAP.md

Usage: `/orbti:remove-project 5`

---

## Milestone Commands

### `/orbti:milestone <name>`
Create a new milestone with projects.

- Guides through milestone definition
- Creates project structure
- Updates ROADMAP.md with milestone grouping

Usage: `/orbti:milestone "v2.0 API Redesign"`

---

### `/orbti:complete-milestone [version]`
Archive milestone, tag, and reorganize roadmap.

- Verifies all projects complete
- Creates git tag (if configured)
- Archives milestone to MILESTONES.md
- Evolves PROJECT.md for next milestone

Usage: `/orbti:complete-milestone`
Usage: `/orbti:complete-milestone v0.3`

---

### `/orbti:observe-milestone`
Explore and articulate vision before starting a milestone.

- Conversational exploration of goals
- Creates milestone context document
- Prepares for `/orbti:milestone`

Usage: `/orbti:observe-milestone`

---

## Pre-Planning Commands

### `/orbti:observe <project>`
Articulate vision and explore approach before planning.

- Conversational discussion of project goals
- Creates CONTEXT.md capturing vision
- Prepares for `/orbti:refine`

Usage: `/orbti:observe 3`
Usage: `/orbti:observe "authentication layer"`

---

### `/orbti:assumptions <project>`
Surface Claude's assumptions about a project before planning.

- Shows what Claude would do if given free rein
- Identifies gaps in understanding
- Prevents misaligned planning

Usage: `/orbti:assumptions 3`

---

### `/orbti:cocreate <topic>`
Research technical options and make decisions before planning a phase.

- Explores options, libraries, and architecture approaches
- Compares alternatives with pros/cons
- Produces COCREATE.md with recommendation and confidence level

Usage: `/orbti:cocreate "authentication patterns"`

---

### `/orbti:consider-issues [source]`
Review deferred issues with codebase context, triage and route.

- Reads deferred issues from STATE.md or specified source
- Analyzes with current codebase context
- Suggests routing: fix now, defer, or close

Usage: `/orbti:consider-issues`

---

## Research Commands

### `/orbti:research <topic>`
Deploy research agents for documentation/web search.

- Spawns subagents for parallel research
- Gathers external documentation
- Creates RESEARCH.md with findings
- Main session vets and reviews results

Usage: `/orbti:research "JWT best practices 2026"`

---

### `/orbti:research-phase <number>`
Research unknowns for a project using subagents.

- Identifies unknowns in project scope
- Deploys research agents
- Synthesizes findings for planning

Usage: `/orbti:research-phase 4`

---

## Specialized Commands

### `/orbti:skills`
Configure skill integrations.

- Creates/updates SPECIAL-FLOWS.md
- Defines required skills per work type
- Skills are enforced at BUILD time

Usage: `/orbti:skills`

---

### `/orbti:config`
View or modify ORBTI configuration.

- Shows current config.md settings
- Allows toggling integrations
- Manages project-level settings

Usage: `/orbti:config`

---

### `/orbti:map-codebase`
Generate codebase map for context.

- Creates structured overview of project
- Identifies key files and patterns
- Useful for research and planning

Usage: `/orbti:map-codebase`

---

## Quality Commands

### `/orbti:test`
Guide manual user acceptance testing of recently built features.

- Generates verification checklist from INTEGRATE.md
- Guides through manual testing
- Records verification results

Usage: `/orbti:test`

---

### `/orbti:refine-fix`
Refine fixes for UAT issues from verify.

- Reads issues identified during verify
- Creates targeted fix refine
- Smaller scope than full project refine

Usage: `/orbti:refine-fix`

---

## Files & Structure

```
.orbti/
├── PROJECT.md           # Project context and value prop
├── ROADMAP.md           # Project breakdown and milestones
├── STATE.md             # Loop position and session state
├── config.md            # Optional integrations config
├── SPECIAL-FLOWS.md     # Optional skill requirements
├── MILESTONES.md        # Completed milestone archive
└── projects/
    ├── 01-foundation/
    │   ├── 01-01-REFINE.md
    │   └── 01-01-INTEGRATE.md
    └── 02-features/
        ├── 02-01-REFINE.md
        └── 02-01-INTEGRATE.md
```

## REFINE.md Structure

```markdown
---
project: 01-foundation
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

<skills>
Required skills from SPECIAL-FLOWS.md
</skills>

<acceptance_criteria>
Given/When/Then format
</acceptance_criteria>

<tasks>
<task type="auto">...</task>
</tasks>

<boundaries>
DO NOT CHANGE, SCOPE LIMITS
</boundaries>

<verification>
Completion checks
</verification>
```

## Task Types

| Type | Use For |
|------|---------|
| `auto` | Fully autonomous execution |
| `checkpoint:decision` | Choices requiring human input |
| `checkpoint:human-verify` | Visual/functional verification |
| `checkpoint:human-action` | Manual steps (rare) |

## Common Workflows

**Starting a new project:**
```
/orbti:init → /orbti:refine → /orbti:build → /orbti:integrate
```

**Checking where you are:**
```
/orbti:progress
```

**Resuming work (new session):**
```
/orbti:resume
```

**Pre-planning exploration:**
```
/orbti:observe 3 → /orbti:assumptions 3 → /orbti:research "topic" → /orbti:refine 3
```



## Key Principles

1. **Loop must complete** - REFINE -> BUILD -> INTEGRATE, no shortcuts
2. **State is tracked** - STATE.md knows where you are
3. **Boundaries are real** - Respect DO NOT CHANGE sections
4. **Acceptance criteria first** - Define done before starting
5. **Skills are enforced** - Required skills block BUILD until loaded

## Getting Help

- Run `/orbti:progress` to see where you are and what to do next
- Read `.orbti/PROJECT.md` for project context
- Read `.orbti/STATE.md` for current position
- Check `.orbti/ROADMAP.md` for project overview

---

*ORBTI Framework v0.4+ | 26 commands | Last updated: 2026-01-29*
</reference>
