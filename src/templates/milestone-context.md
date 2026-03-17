# Milestone Context Template

Template for `.orbit/projects/{name}/MILESTONE-CONTEXT.md` — temporary handoff from cocreate-milestone to create-milestone.

**Purpose:** Persist milestone discussion context across `/clear` boundaries. This is a handoff artifact, not permanent documentation.

---

## File Template

```markdown
# Milestone Context

**Generated:** {{DATE}}
**Status:** Ready for /orbit:create-milestone

## Features to Build

[Features identified during discussion — the substance of this milestone]

- **{{FEATURE_1}}**: {{DESCRIPTION}}
- **{{FEATURE_2}}**: {{DESCRIPTION}}
- **{{FEATURE_3}}**: {{DESCRIPTION}}

## Scope

**Suggested name:** v{{VERSION}} {{THEME_NAME}}
**Estimated projects:** {{PROJECT_COUNT}}
**Focus:** {{ONE_SENTENCE_THEME}}

## Project Mapping

[How features map to projects — rough breakdown]

| Project | Focus | Features |
|---------|-------|----------|
| {{NEXT_PROJECT}} | {{FOCUS}} | {{FEATURES}} |
| {{NEXT_PROJECT+1}} | {{FOCUS}} | {{FEATURES}} |
| {{NEXT_PROJECT+2}} | {{FOCUS}} | {{FEATURES}} |

## Constraints

[Any constraints or boundaries identified during discussion]

- {{CONSTRAINT_1}}
- {{CONSTRAINT_2}}

## Additional Context

[Anything else captured during discussion that informs the milestone]

{{NOTES}}

---

*This file is temporary. It will be deleted after /orbit:create-milestone creates the milestone.*
*Do NOT commit this file to version control.*
```

---

## Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    MILESTONE CONTEXT LIFECYCLE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. /orbit:cocreate-milestone                                      │
│     └─> Explores features, scope, constraints                    │
│     └─> Creates .orbit/projects/{name}/MILESTONE-CONTEXT.md      │
│                                                                  │
│  2. /clear (safe now)                                            │
│     └─> Context persisted in file                                │
│     └─> Session can be cleared                                   │
│                                                                  │
│  3. /orbit:create-milestone                                       │
│     └─> Reads .orbit/projects/{name}/MILESTONE-CONTEXT.md        │
│     └─> Uses context to populate ROADMAP.md                      │
│     └─> Deletes MILESTONE-CONTEXT.md after success               │
│                                                                  │
│  Result: Clean handoff across session boundaries                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Points

1. **Created by:** `cocreate-milestone` command at end of discussion
2. **Consumed by:** `create-milestone` command
3. **Deleted after:** Milestone successfully created
4. **Not committed:** Exclude from version control (temporary artifact)

---

## Content Guidelines

### What to Include

| Section | Include | Purpose |
|---------|---------|---------|
| Features | User-facing capabilities to build | Defines milestone substance |
| Scope | Version number, theme, phase count | Frames the work |
| Phase Mapping | Rough feature-to-phase breakdown | Guides create-milestone |
| Constraints | Technical/business limits | Prevents scope creep |
| Additional Context | Discussion notes, decisions | Preserves conversation |

### What NOT to Include

| Avoid | Why |
|-------|-----|
| Technical implementation details | That comes during phase planning |
| Detailed phase specifications | create-milestone handles that |
| Code architecture decisions | Research phase work |
| Exhaustive task lists | LOOP.md level of detail |

**Key principle:** This captures the *what* and *why*, not the *how*.

---

## Integration with ORBIT Workflow

### Before cocreate-milestone

```
STATE.md shows: Previous milestone complete
ROADMAP.md shows: Phases complete, next milestone undefined
```

### After cocreate-milestone

```
.orbit/projects/{name}/MILESTONE-CONTEXT.md: Created with discussion output
STATE.md: Updated to note milestone discussion complete
```

### After create-milestone

```
MILESTONE-CONTEXT.md: Deleted
ROADMAP.md: Updated with new milestone and projects
STATE.md: Updated with new project position
```

---

## Example

```markdown
# Milestone Context

**Generated:** 2026-01-29
**Status:** Ready for /orbit:create-milestone

## Features to Build

- **Milestone lifecycle commands**: Create, complete, and discuss milestones
- **Roadmap modification commands**: Add and remove projects dynamically
- **Milestone templates**: Entry format, archive format, context handoff

## Scope

**Suggested name:** v0.3 Roadmap & Milestone Management
**Estimated projects:** 3
**Focus:** Complete milestone management tooling for ORBIT framework

## Project Mapping

| Project | Focus | Features |
|---------|-------|----------|
| 9 | Templates | MILESTONES.md, archive, context templates |
| 10 | Workflows | cocreate-milestone, complete-milestone, add-project |
| 11 | Commands | CLI commands for milestone operations |

## Constraints

- Templates first, then workflows, then commands (dependency order)
- Keep commands simple — complex logic in workflows

## Additional Context

- Discussed need for milestone grouping in ROADMAP.md
- User preference for tabular layouts over bullet lists
- Consider optional SonarQube scan before milestone completion

---

*This file is temporary. It will be deleted after /orbit:create-milestone creates the milestone.*
*Do NOT commit this file to version control.*
```
