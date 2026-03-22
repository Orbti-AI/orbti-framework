# INTEGRATE.md Template

Template for `.orbti/projects/{name}/{refine}-INTEGRATE.md` — refine completion documentation.

**Purpose:** Document what was built, decisions made, deviations from refine, and readiness for next project.

---

## File Template

```markdown
---
phase: name
refine: NN
subsystem: [primary category: auth, payments, ui, api, database, infra, testing]
tags: [searchable tech: jwt, stripe, react, postgres, prisma]

# Dependency graph
requires:
  - phase: [prior phase this depends on]
    provides: [what that phase built that this uses]
provides:
  - [what this refine built/delivered]
  - [another deliverable]
affects: [phase names or keywords that will need this context]

# Tech tracking
tech-stack:
  added: [libraries/tools added]
  patterns: [architectural patterns established]

key-files:
  created: [important files created]
  modified: [important files modified]

key-decisions:
  - "Decision 1: [brief]"
  - "Decision 2: [brief]"

patterns-established:
  - "Pattern 1: [description]"
  - "Pattern 2: [description]"

# Metrics
duration: Xmin
started: YYYY-MM-DDTHH:MM:SSZ
completed: YYYY-MM-DDTHH:MM:SSZ
---

# Phase [X] Refine [Y]: [Name] Summary

**[Substantive one-liner describing outcome - what actually shipped]**

## Performance

| Metric | Value |
|--------|-------|
| Duration | [time] |
| Started | [ISO timestamp] |
| Completed | [ISO timestamp] |
| Tasks | [N] completed |
| Files modified | [N] |

## Acceptance Criteria Results

| Criterion | Status | Notes |
|-----------|--------|-------|
| AC-1: [Name] | Pass / Fail | [Details if needed] |
| AC-2: [Name] | Pass / Fail | [Details] |
| AC-3: [Name] | Pass / Fail | [Details] |

## Accomplishments

- [Most important outcome - specific, substantive]
- [Second key accomplishment]
- [Third if applicable]

## Task Commits

Each task committed atomically:

| Task | Commit | Type | Description |
|------|--------|------|-------------|
| Task 1: [name] | `abc123f` | feat | [What was done] |
| Task 2: [name] | `def456g` | feat | [What was done] |
| Task 3: [name] | `hij789k` | test | [What was done] |

Refine metadata: `lmn012o` (docs: complete refine)

## Files Created/Modified

| File | Change | Purpose |
|------|--------|---------|
| `path/to/file.ts` | Created | [What it does] |
| `path/to/another.ts` | Modified | [What changed] |

## Decisions Made

[Key decisions with brief rationale]

| Decision | Rationale | Impact |
|----------|-----------|--------|
| [What was decided] | [Why] | [Effect on future work] |

Or: "None - followed refine as specified"

## Deviations from Refine

### Summary

| Type | Count | Impact |
|------|-------|--------|
| Auto-fixed | [N] | [Brief assessment] |
| Scope additions | [N] | [Brief assessment] |
| Deferred | [N] | Logged to issues |

**Total impact:** [Brief assessment - e.g., "Essential fixes, no scope creep"]

### Auto-fixed Issues

**1. [Category] [Brief description]**
- **Found during:** Task [N] ([task name])
- **Issue:** [What was wrong]
- **Fix:** [What was done]
- **Files:** [Affected files]
- **Verification:** [How verified]
- **Commit:** [hash] (part of task commit)

[Repeat for each auto-fix]

### Deferred Items

Logged to issue tracker for future consideration:
- [Issue ID]: [Brief description] (discovered in Task [N])

Or: "None - refine executed exactly as written"

## Issues Encountered

[Problems during execution and how resolved]

| Issue | Resolution |
|-------|------------|
| [Problem] | [How solved] |

Or: "None"

## Next Phase Readiness

**Ready:**
- [What's ready for next phase]
- [Foundation established]

**Concerns:**
- [Potential issues for future phases]
- [Technical debt introduced]

**Blockers:**
- [Anything blocking next phase] or "None"

---
*Phase: name, Refine: NN*
*Completed: [YYYY-MM-DD]*
```

---

## Section Specifications

### Frontmatter
**Purpose:** Machine-readable metadata for automatic context assembly.

| Field | Purpose |
|-------|---------|
| `phase`, `refine` | Identification |
| `subsystem` | Primary categorization (auth, api, ui, etc.) |
| `tags` | Searchable tech keywords |
| `requires` | What prior phases this depended on |
| `provides` | What this refine delivers |
| `affects` | Future phases that need this context |
| `tech-stack` | Libraries added, patterns established |
| `key-files` | Important files for @context references |
| `key-decisions` | Quick-reference decisions (detail in body) |
| `patterns-established` | Conventions future phases should follow |
| `duration`, `started`, `completed` | Performance metrics |

### One-liner Requirements

**Must be substantive:**

| Good | Bad |
|------|-----|
| "JWT auth with refresh rotation using jose library" | "Phase complete" |
| "Prisma schema with User, Session, Product models" | "Authentication implemented" |
| "Dashboard with real-time metrics via SSE" | "All tasks done" |

The one-liner should tell someone what actually shipped.

### Acceptance Criteria Results
**Purpose:** Verify all AC from REFINE.md were met.
**Contains:** Each AC with pass/fail status and notes.
**Requirement:** All must pass for refine to be complete.

### Deviations Section
**Purpose:** Document unplanned work and its handling.

**Auto-fixed:** Issues discovered and resolved during execution.
- Include: category, what was wrong, fix, verification
- Must be essential fixes, not scope creep

**Deferred:** Items logged for future consideration.
- Include: issue ID, brief description, origin task

**Note:** Separate from "Issues Encountered" which documents problems with planned work.

### Next Phase Readiness
**Purpose:** Prepare context for next REFINE phase.

**Ready:** What's available for next phase to build on.
**Concerns:** Potential issues to watch for.
**Blockers:** Anything preventing next phase (usually "None").
