# Milestone Archive Template

Template for milestone archives at `.orbit/milestones/v{VERSION}-{NAME}.md`.

**Purpose:** Preserve complete phase details for completed milestones. Created by complete-milestone workflow when a milestone ships.

---

## File Template

```markdown
# Milestone v{{VERSION}}: {{MILESTONE_NAME}}

**Status:** ✅ SHIPPED {{DATE}}
**Phases:** {{PHASE_START}}-{{PHASE_END}}
**Total Loops:** {{LOOP_COUNT}}

## Overview

{{MILESTONE_DESCRIPTION}}

## Projects

{{#PROJECTS}}
### Project {{PROJECT_NUM}}: {{PROJECT_NAME}}

**Goal:** {{PROJECT_GOAL}}
**Depends on:** {{DEPENDS_ON}}
**Loops:** {{LOOP_COUNT}} completed

Plans:
- [x] {{PROJECT_NUM}}-01: {{LOOP_DESCRIPTION}}
- [x] {{PROJECT_NUM}}-02: {{LOOP_DESCRIPTION}}
[... all plans ...]

**Details:**
{{PROJECT_SCOPE_FROM_ROADMAP}}
{{/PROJECTS}}

{{#DECIMAL_PROJECTS}}
### Project {{PROJECT_NUM}}: {{PROJECT_NAME}} [INSERTED]

**Goal:** {{PROJECT_GOAL}}
**Depends on:** {{DEPENDS_ON}}
**Reason:** {{INSERTION_REASON}}

Plans:
- [x] {{PROJECT_NUM}}-01: {{LOOP_DESCRIPTION}}
{{/DECIMAL_PROJECTS}}

---

## Milestone Summary

**Decimal Projects:**
{{#HAS_DECIMAL_PROJECTS}}
- Project {{PROJECT_NUM}}: {{PROJECT_NAME}} (inserted after Project {{PARENT_PROJECT}} for {{REASON}})
{{/HAS_DECIMAL_PROJECTS}}
{{^HAS_DECIMAL_PROJECTS}}
None
{{/HAS_DECIMAL_PROJECTS}}

**Key Decisions:**
{{#DECISIONS}}
- {{DECISION}} (Rationale: {{RATIONALE}})
{{/DECISIONS}}

**Issues Resolved:**
{{#ISSUES_RESOLVED}}
- {{ISSUE}}
{{/ISSUES_RESOLVED}}

**Issues Deferred:**
{{#ISSUES_DEFERRED}}
- {{ISSUE}} (deferred to: {{DEFERRED_TO}})
{{/ISSUES_DEFERRED}}

**Technical Debt Incurred:**
{{#TECH_DEBT}}
- {{DEBT_ITEM}} (address in: {{WHEN}})
{{/TECH_DEBT}}

---

*For current project status, see [ROADMAP.md](../ROADMAP.md) and [STATE.md](../STATE.md)*
```

---

## Placeholder Reference

| Placeholder | Source | Example |
|-------------|--------|---------|
| `{{VERSION}}` | Milestone version | `0.2` |
| `{{MILESTONE_NAME}}` | Milestone theme | `Session Continuity` |
| `{{DATE}}` | Ship date | `2026-01-28` |
| `{{PHASE_START}}` | First project number | `7` |
| `{{PHASE_END}}` | Last project number | `8.7` |
| `{{LOOP_COUNT}}` | Total plans | `5` |
| `{{MILESTONE_DESCRIPTION}}` | One-sentence summary | From MILESTONES.md entry |
| `{{PROJECT_NUM}}` | Project number | `7` or `8.5` |
| `{{PROJECT_NAME}}` | Project name | `Session Handoff` |
| `{{PROJECT_GOAL}}` | Project goal | From ROADMAP.md |
| `{{DEPENDS_ON}}` | Project dependencies | `Project 6` |
| `{{LOOP_DESCRIPTION}}` | Plan brief | From ROADMAP.md or SUMMARY |
| `{{PROJECT_SCOPE_FROM_ROADMAP}}` | Scope bullets | From ROADMAP.md |
| `{{INSERTION_REASON}}` | Why decimal inserted | `Urgent quality fix` |
| `{{DECISION}}` | Decision made | From STATE.md Decisions |
| `{{RATIONALE}}` | Why decided | From STATE.md |

---

## Filename Convention

**Pattern:** `.orbit/milestones/v{VERSION}-{NAME}.md`

**Rules:**
- VERSION: Numeric version without `v` prefix in filename → `0.2`, `1.0`, `2.0`
- NAME: Kebab-case milestone theme → `session-continuity`, `core-loop`, `mvp`

**Examples:**
- `v0.1-core-loop.md`
- `v0.2-session-continuity.md`
- `v1.0-mvp.md`
- `v2.0-redesign.md`

---

## Usage Guidelines

### When to create archives

Archives are created by the `complete-milestone` workflow when:
- All phases in a milestone are complete
- Milestone is officially shipped
- Before planning begins on next milestone

### How to fill template

The workflow populates from:
1. **ROADMAP.md** — Phase details, goals, scope, plan lists
2. **STATE.md** — Decisions from Accumulated Context section
3. **MILESTONES.md** — Overview description
4. **SUMMARY files** — Plan completion details

### After archiving

1. Update ROADMAP.md — Collapse milestone in `<details>` tag
2. Update STATE.md — Clear completed phase context
3. Continue numbering — Never restart phase numbers at 01

---

## Example

```markdown
# Milestone v0.2: Session Continuity

**Status:** ✅ SHIPPED 2026-01-28
**Phases:** 7-8.7
**Total Plans:** 5

## Overview

Pause/resume workflow with handoff files and enhanced STATE.md session tracking.

## Projects

### Project 7: Session Handoff

**Goal:** Create pause/resume workflow for session breaks
**Depends on:** Project 6
**Plans:** 1 completed

Plans:
- [x] 07-01: Handoff file generation and resume workflow

**Details:**
- HANDOFF-*.md template for context capture
- Resume command for context restoration
- Session continuity section in STATE.md

### Project 8.5: Quality Extensions [INSERTED]

**Goal:** Code quality tooling integration
**Depends on:** Project 8
**Reason:** Discovered need during Project 8 review

Plans:
- [x] 08.5-01: SonarQube integration (optional)

### Project 8.6: Codebase CLAUDE.md [INSERTED]

**Goal:** Project-level Claude Code configuration
**Depends on:** Project 8.5
**Reason:** Improve agent context for ORBIT projects

Plans:
- [x] 08.6-01: CLAUDE.md template for codebase guidance

### Project 8.7: SonarQube Integration [INSERTED]

**Goal:** Automated code quality checks
**Depends on:** Project 8.6
**Reason:** Complete quality tooling suite

Plans:
- [x] 08.7-01: SonarQube project setup and config

---

## Milestone Summary

**Decimal Projects:**
- Project 8.5: Quality Extensions (inserted after Project 8 for code quality tooling)
- Project 8.6: Codebase CLAUDE.md (inserted after Project 8.5 for agent context)
- Project 8.7: SonarQube Integration (inserted after Project 8.6 for quality checks)

**Key Decisions:**
- SonarQube is optional integration (Rationale: Not all projects need code quality tooling)
- Minimize subagents for dev work (Rationale: ~70% quality, high token cost)

**Issues Resolved:**
- Session context lost after /clear
- No way to pause and resume work

**Issues Deferred:**
- None

**Technical Debt Incurred:**
- None

---

*For current project status, see [ROADMAP.md](../ROADMAP.md) and [STATE.md](../STATE.md)*
```
