# STATE.md Template

Template for `.orbit/STATE.md` — the project's living memory.

**Purpose:** Single source of truth for current position, all-projects overview, and session continuity.

---

## File Template

```markdown
# Project State

## Projects Overview

All projects in the current milestone — updated after every loop completion.

| # | Project | Loops | Status | Loop Position |
|---|---------|-------|--------|---------------|
| 01-[name] | [Description] | 0/N | ○ Pending | ○ ○ ○ |
| 02-[name] | [Description] | 1/N | ⏸ Paused | ✓ ◉ ○ |
| 03-[name] | [Description] | 1/N | 🔵 In Progress | ◉ ○ ○ |
| 04-[name] | [Description] | N/N | ✅ Complete | ✓ ✓ ✓ |

**Milestone progress:** [X] of [Y] projects complete

---

## Current Focus

**Project:** [N] — [Name]
**Plan:** [A] of [B]
**Status:** [Ready to refine | Planning | Building | Integrating | Complete | Blocked]
**Last activity:** [YYYY-MM-DD HH:MM] — [What happened]

Loop position:
```
REFINE ──▶ BUILD ──▶ INTEGRATE
  ◉        ○        ○     [Planning]
  ✓        ◉        ○     [Building]
  ✓        ✓        ◉     [Integrating]
  ✓        ✓        ✓     [Complete - ready for next REFINE]
```

## Accumulated Context

### Decisions

| Decision | Project | Impact |
|----------|---------|--------|
| [Decision summary] | [Project N] | [Ongoing effect] |

### Deferred Issues

| Issue | Origin | Effort | Revisit |
|-------|--------|--------|---------|
| [Brief description] | [Project N] | [S/M/L] | [When to reconsider] |

### Blockers

| Blocker | Impact | Resolution Path |
|---------|--------|-----------------|
| [Description] | [What's blocked] | [How to resolve] |

## Session Continuity

Last session: [YYYY-MM-DD HH:MM]
Stopped at: [Description of last completed action]
Next action: [Exactly ONE thing to do next]
Resume file: .orbit/projects/[current-project]/HANDOFF-[date].md

---
*STATE.md — Updated after every significant action*
*Size target: <100 lines (digest, not archive)*
```

---

## Section Specifications

### Projects Overview
**Purpose:** Dashboard of all projects in the milestone — visible at a glance without reading ROADMAP.md.

**Status values:**
- `○ Pending` — not started
- `🔵 In Progress` — active project (current session focus)
- `⏸ Paused` — mid-loop, switched to another project
- `✅ Complete` — all loops done, INTEGRATE complete

**Loop Position column** (compact):
- `○ ○ ○` — not started
- `◉ ○ ○` — REFINE active
- `✓ ◉ ○` — BUILD active
- `✓ ✓ ◉` — INTEGRATE active
- `✓ ✓ ✓` — loop complete (repeats per loop count)

**Update:** After every INTEGRATE completion and every project transition.

**Row format:**
```
| 01-auth | JWT authentication | 2/3 | 🔵 In Progress | ✓ ✓ ✓ / ✓ ✓ ✓ / ✓ ◉ ○ |
```
When a project has multiple loops, show each loop's position separated by `/`.

### Current Focus
**Purpose:** Deep view of the active project — what's happening right now.

**Contains:**
- Project number and name
- Plan number within project
- Status and last activity
- Visual loop position diagram

**Update:** At every loop state change (REFINE → BUILD → INTEGRATE → complete).

### Accumulated Context
**Purpose:** Digest of decisions, issues, blockers relevant to current work.

**Decisions:** Keep only 3-5 recent entries. Full log in PROJECT.md.
**Deferred Issues:** Items logged but not yet addressed.
**Blockers:** Active obstacles only — remove when resolved.

**Update:** After INTEGRATE reconciliation.

### Session Continuity
**Purpose:** Enable instant resumption — one read, one action.

**Next action:** Must be exactly ONE thing (not a list). If unsure, point to `/orbit:progress`.

**Update:** At end of every session or after `/orbit:pause`.

---

## How the Overview gets populated

**At `/orbit:init`:**
- Add a row per project defined in ROADMAP.md
- All rows start as `○ Pending | ○ ○ ○`

**At `/orbit:refine` (plan start):**
- Update project row: Status → `🔵 In Progress`
- Update loop position column for active plan

**At `/orbit:build` (tasks running):**
- Update loop position: `◉ ○ ○` → `✓ ◉ ○`

**At `/orbit:integrate` (loop closes):**
- Update loop position: `✓ ✓ ◉` → `✓ ✓ ✓`
- Increment completed loop count: `1/3` → `2/3`

**At project transition (all loops done):**
- Update project row: Status → `✅ Complete`
- Update milestone progress counter

---

## Size Constraint

**Target:** Under 100 lines.

STATE.md is a DIGEST, not an archive:
- Keep only 3-5 recent decisions (full log in PROJECT.md)
- Keep only active blockers (remove resolved)
- Projects Overview is compact by design — one row per project

**Goal:** "Read once, know where everything stands."

---

## Lifecycle

**Creation:** After ROADMAP.md created (during `/orbit:init`)
- Build Projects Overview table from ROADMAP.md projects
- All rows start as Pending
- Set Current Focus to "Project 1 — Ready to refine"

**Reading:** First step of EVERY workflow
- Load before any REFINE, BUILD, or INTEGRATE

**Writing:** After every significant action
- REFINE start: mark project In Progress, update loop position
- BUILD complete: update loop position in Overview
- INTEGRATE: close loop, increment count, update overview row
- Project transition: mark Complete, update milestone counter
- Session end: update Session Continuity
