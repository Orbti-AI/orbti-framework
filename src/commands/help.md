---
name: orbit:help
description: Show available ORBIT commands and usage guide
---

<model>haiku</model>

<objective>
Display the complete ORBIT command reference.

Output ONLY the reference content below. Do NOT add:

- Project-specific analysis
- Git status or file context
- Next-step suggestions
- Any commentary beyond the reference
</objective>

<reference>
# ORBIT Command Reference

**ORBIT** (Observe, Refine, Build, Integrate, Test) is a structured AI-assisted development framework for Claude Code.

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

**Never skip INTEGRATE.** Every plan needs a summary.

## Quick Start

1. `/orbit:init` - Initialize ORBIT in your project
2. `/orbit:refine` - Create a plan for your work
3. `/orbit:build` - Execute the approved refine
4. `/orbit:integrate` - Close the loop with summary

## Commands Overview

| Category | Commands |
|----------|----------|
| Core Loop | refine, build, integrate, test, help, status |
| Session | pause, resume, progress, handoff |
| Roadmap | add-project, remove-project |
| Milestone | milestone, complete-milestone, cocreate-milestone |
| Pre-Planning | cocreate, assumptions, observe, consider-issues |
| Research | research, research-phase |
| Specialized | flows, config, map-codebase |
| Quality | test, plan-fix |

---

## Core Loop Commands

### `/orbit:init`
Initialize ORBIT in a project.

- Creates `.orbit/` directory structure
- Creates PROJECT.md, STATE.md, ROADMAP.md
- Prompts for project context and phases
- Optionally configures integrations (SonarQube, etc.)

Usage: `/orbit:init`

---

### `/orbit:refine [project]`
Enter REFINE phase - create an executable plan.

- Reads current state from STATE.md
- Creates LOOP.md with tasks, acceptance criteria, boundaries
- Populates skills section from SPECIAL-FLOWS.md (if configured)
- Updates loop position

Usage: `/orbit:refine` (auto-detects next project)
Usage: `/orbit:refine 3` (specific project)

---

### `/orbit:build [plan-path]`
Execute an approved LOOP.md file.

- **Blocks if required skills not loaded** (from SPECIAL-FLOWS.md)
- Validates plan exists and hasn't been executed
- Executes tasks sequentially
- Handles checkpoints (decision, human-verify, human-action)
- Reports completion and prompts for INTEGRATE

Usage: `/orbit:build`
Usage: `/orbit:build .orbit/projects/01-foundation/01-01-LOOP.md`

---

### `/orbit:integrate [plan-path]`
Reconcile refine vs actual and close the loop.

- Creates INTEGRATE.md documenting what was built
- Audits skill invocations (if SPECIAL-FLOWS.md configured)
- Records decisions made, deferred issues
- Updates STATE.md with loop closure
- **Required** - never skip this step

Usage: `/orbit:integrate`
Usage: `/orbit:integrate .orbit/projects/01-foundation/01-01-LOOP.md`

---

### `/orbit:help`
Show this command reference.

Usage: `/orbit:help`

---

### `/orbit:status` *(deprecated)*
> Use `/orbit:progress` instead.

Shows current loop position. Deprecated in favor of `/orbit:progress` which provides better routing.

---

## Session Commands

### `/orbit:pause [reason]`
Create handoff file and prepare for session break.

- Creates HANDOFF.md with complete context
- Updates STATE.md session continuity section
- Designed for context limits or multi-session work

Usage: `/orbit:pause`
Usage: `/orbit:pause "switching to other project"`

---

### `/orbit:resume [handoff-path]`
Restore context from handoff and continue work.

- Reads STATE.md and any HANDOFF files
- Determines current loop position
- Suggests exactly ONE next action
- Archives consumed handoffs

Usage: `/orbit:resume`
Usage: `/orbit:resume .orbit/HANDOFF-context.md`

---

### `/orbit:progress [context]`
Smart status with routing - suggests ONE next action.

- Shows milestone and phase progress visually
- Displays current loop position
- Suggests exactly ONE next action (prevents decision fatigue)
- Accepts optional context to tailor suggestion
- Warns about context limits

Usage: `/orbit:progress`
Usage: `/orbit:progress "I only have 30 minutes"`

---

### `/orbit:handoff [context]`
Generate comprehensive session handoff document.

- Creates detailed handoff for complex session breaks
- Captures decisions, progress, blockers, next steps
- More thorough than `/orbit:pause`

Usage: `/orbit:handoff`
Usage: `/orbit:handoff "phase10-audit"`

---

## Roadmap Commands

### `/orbit:add-project <description>`
Append a new project to the roadmap.

- Adds project to end of ROADMAP.md
- Updates project numbering
- Records in STATE.md decisions

Usage: `/orbit:add-project "API Authentication Layer"`

---

### `/orbit:remove-project <number>`
Remove a future (not started) project from roadmap.

- Cannot remove completed or in-progress projects
- Renumbers subsequent projects
- Updates ROADMAP.md

Usage: `/orbit:remove-project 5`

---

## Milestone Commands

### `/orbit:milestone <name>`
Create a new milestone with projects.

- Guides through milestone definition
- Creates project structure
- Updates ROADMAP.md with milestone grouping

Usage: `/orbit:milestone "v2.0 API Redesign"`

---

### `/orbit:complete-milestone [version]`
Archive milestone, tag, and reorganize roadmap.

- Verifies all projects complete
- Creates git tag (if configured)
- Archives milestone to MILESTONES.md
- Evolves PROJECT.md for next milestone

Usage: `/orbit:complete-milestone`
Usage: `/orbit:complete-milestone v0.3`

---

### `/orbit:cocreate-milestone`
Explore and articulate vision before starting a milestone.

- Conversational exploration of goals
- Creates milestone context document
- Prepares for `/orbit:milestone`

Usage: `/orbit:cocreate-milestone`

---

## Pre-Planning Commands

### `/orbit:cocreate <project>`
Articulate vision and explore approach before planning.

- Conversational discussion of project goals
- Creates CONTEXT.md capturing vision
- Prepares for `/orbit:refine`

Usage: `/orbit:cocreate 3`
Usage: `/orbit:cocreate "authentication layer"`

---

### `/orbit:assumptions <project>`
Surface Claude's assumptions about a project before planning.

- Shows what Claude would do if given free rein
- Identifies gaps in understanding
- Prevents misaligned planning

Usage: `/orbit:assumptions 3`

---

### `/orbit:observe <topic>`
Research technical options and make decisions before planning a phase.

- Explores options, libraries, and architecture approaches
- Compares alternatives with pros/cons
- Produces OBSERVE.md with recommendation and confidence level

Usage: `/orbit:observe "authentication patterns"`

---

### `/orbit:consider-issues [source]`
Review deferred issues with codebase context, triage and route.

- Reads deferred issues from STATE.md or specified source
- Analyzes with current codebase context
- Suggests routing: fix now, defer, or close

Usage: `/orbit:consider-issues`

---

## Research Commands

### `/orbit:research <topic>`
Deploy research agents for documentation/web search.

- Spawns subagents for parallel research
- Gathers external documentation
- Creates RESEARCH.md with findings
- Main session vets and reviews results

Usage: `/orbit:research "JWT best practices 2026"`

---

### `/orbit:research-phase <number>`
Research unknowns for a project using subagents.

- Identifies unknowns in project scope
- Deploys research agents
- Synthesizes findings for planning

Usage: `/orbit:research-phase 4`

---

## Specialized Commands

### `/orbit:flows`
Configure specialized workflow integrations.

- Creates/updates SPECIAL-FLOWS.md
- Defines required skills per work type
- Skills are enforced at BUILD time

Usage: `/orbit:flows`

---

### `/orbit:config`
View or modify ORBIT configuration.

- Shows current config.md settings
- Allows toggling integrations
- Manages project-level settings

Usage: `/orbit:config`

---

### `/orbit:map-codebase`
Generate codebase map for context.

- Creates structured overview of project
- Identifies key files and patterns
- Useful for research and planning

Usage: `/orbit:map-codebase`

---

## Quality Commands

### `/orbit:test`
Guide manual user acceptance testing of recently built features.

- Generates verification checklist from INTEGRATE.md
- Guides through manual testing
- Records verification results

Usage: `/orbit:test`

---

### `/orbit:refine-fix`
Refine fixes for UAT issues from verify.

- Reads issues identified during verify
- Creates targeted fix refine
- Smaller scope than full project plan

Usage: `/orbit:refine-fix`

---

## Files & Structure

```
.orbit/
├── PROJECT.md           # Project context and value prop
├── ROADMAP.md           # Project breakdown and milestones
├── STATE.md             # Loop position and session state
├── config.md            # Optional integrations config
├── SPECIAL-FLOWS.md     # Optional skill requirements
├── MILESTONES.md        # Completed milestone archive
└── projects/
    ├── 01-foundation/
    │   ├── 01-01-LOOP.md
    │   └── 01-01-INTEGRATE.md
    └── 02-features/
        ├── 02-01-LOOP.md
        └── 02-01-INTEGRATE.md
```

## LOOP.md Structure

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
/orbit:init → /orbit:refine → /orbit:build → /orbit:integrate
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
/orbit:cocreate 3 → /orbit:assumptions 3 → /orbit:research "topic" → /orbit:refine 3
```



## Key Principles

1. **Loop must complete** - REFINE -> BUILD -> INTEGRATE, no shortcuts
2. **State is tracked** - STATE.md knows where you are
3. **Boundaries are real** - Respect DO NOT CHANGE sections
4. **Acceptance criteria first** - Define done before starting
5. **Skills are enforced** - Required skills block BUILD until loaded

## Getting Help

- Run `/orbit:progress` to see where you are and what to do next
- Read `.orbit/PROJECT.md` for project context
- Read `.orbit/STATE.md` for current position
- Check `.orbit/ROADMAP.md` for project overview

---

*ORBIT Framework v0.4+ | 26 commands | Last updated: 2026-01-29*
</reference>
