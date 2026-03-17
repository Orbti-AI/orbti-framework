# CONTEXT.md Template

Template for `.orbit/projects/{NN}-{name}/CONTEXT.md` - project discussion context handoff.

**Purpose:** Captures user's vision, goals, and approach before planning. Created by `/orbit:cocreate`, consumed by `/orbit:refine`.

---

## File Template

```markdown
# Project Context

**Project:** {NN} — {project_name}
**Generated:** {date}
**Status:** Ready for planning | Needs more discussion

## Goals

{user's goals as bullets}
- Goal 1: [description]
- Goal 2: [description]
- Goal 3: [description]

## Approach

{user's approach notes}
- Pattern/library preferences
- Technical direction
- Constraints mentioned

## Constraints

{any limitations or requirements}
- Constraint 1
- Constraint 2

## Open Questions

{items still unclear or needing research}
- Question 1
- Question 2

## Additional Context

{any other notes from discussion}

---

*This file is temporary. It informs planning but is not required.*
*Created by /orbit:cocreate, consumed by /orbit:refine.*
```

---

## Field Descriptions

| Field | Purpose |
|-------|---------|
| `Project` | Project number and name from ROADMAP.md |
| `Generated` | Date context was created |
| `Status` | Whether context is complete or needs more discussion |
| `Goals` | User's objectives for the project |
| `Approach` | Technical direction, patterns, preferences |
| `Constraints` | Limitations or requirements |
| `Open Questions` | Items needing research or decisions |
| `Additional Context` | Other notes from discussion |

---

## Usage Pattern

1. User runs `/orbit:cocreate <project>`
2. Discussion workflow explores goals and approach
3. CONTEXT.md created in project directory
4. `/orbit:refine` reads CONTEXT.md to inform planning
5. CONTEXT.md remains for reference (not deleted)

---

## Relationship to Other Files

| File | Relationship |
|------|--------------|
| ROADMAP.md | Source of project definition |
| CONTEXT.md | Discussion output (this file) |
| LOOP.md | Consumes context for planning |
| RESEARCH.md | May inform context or result from questions |
