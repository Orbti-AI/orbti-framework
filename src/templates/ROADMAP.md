# ROADMAP.md Template

Template for `.orbti/ROADMAP.md` — the project's phase structure.

**Purpose:** Define milestones and projects. Provides structure, not detailed tasks.

---

## Milestone Status Legend

| Emoji | Status | Meaning |
|-------|--------|---------|
| ✅ | Shipped | Milestone complete and archived |
| 🚧 | In Progress | Active development |
| 📋 | Planned | Defined but not started |

---

## File Template (Initial v1.0)

```markdown
# Roadmap: [Project Name]

## Overview

[One paragraph describing the journey from start to finish]

## Current Milestone

**[Milestone Name]** ([version])
Status: [Not started | In progress | Complete]
Projects: [X] of [Y] complete

## Projects

Projects execute in list order. Add new projects at the end or between existing ones as needed.

| Project | Refines | Status | Completed |
|---------|---------|--------|-----------|
| [name] | [N] | Not started | - |
| [name] | [N] | Not started | - |
| [name] | [N] | Not started | - |
| [name] | [N] | Not started | - |

## Project Details

### [name]

**Goal:** [What this project delivers - specific outcome]
**Depends on:** Nothing (first project)
**Research:** [Unlikely | Likely] ([reason])

**Scope:**
- [Deliverable 1]
- [Deliverable 2]
- [Deliverable 3]

**Refines:**
- [ ] 01: [Brief description]
- [ ] 02: [Brief description]
- [ ] 03: [Brief description]

### [name]

**Goal:** [What this project delivers]
**Depends on:** [prior project name] ([specific dependency])
**Research:** [Unlikely | Likely] ([reason])
**Research topics:** [If Likely - what needs investigating]

**Scope:**
- [Deliverable 1]
- [Deliverable 2]

**Refines:**
- [ ] 01: [Brief description]
- [ ] 02: [Brief description]

### [name]

**Goal:** [What this project delivers]
**Depends on:** [prior project name] ([specific dependency])
**Research:** [Unlikely | Likely]

**Scope:**
- [Deliverable 1]
- [Deliverable 2]

**Refines:**
- [ ] 01: [Brief description]
- [ ] 02: [Brief description]

### [name]

**Goal:** [What this project delivers]
**Depends on:** [prior project name]
**Research:** Unlikely (internal patterns)

**Scope:**
- [Deliverable 1]

**Refines:**
- [ ] 01: [Brief description]

---
*Roadmap created: [YYYY-MM-DD]*
*Last updated: [YYYY-MM-DD]*
```

---

## File Template (After v1.0 Ships)

After completing first milestone, reorganize with milestone groupings:

```markdown
# Roadmap: [Project Name]

## Milestones

| Version | Name | Phases | Status | Completed |
|---------|------|--------|--------|-----------|
| v1.0 | MVP | 1-4 | ✅ Shipped | YYYY-MM-DD |
| v1.1 | [Name] | 5-6 | 🚧 In Progress | - |
| v2.0 | [Name] | 7-10 | 📋 Planned | - |

## 🚧 Active Milestone: v1.1 [Name]

**Goal:** [What v1.1 delivers]
**Status:** Project [X] of [Y]
**Progress:** [██████░░░░] 60%

### [name]

**Goal:** [What this project delivers]
**Depends on:** [prior project name]

**Refines:**
- [x] 01: [Completed refine]
- [ ] 02: [In progress or pending]

### [name]

**Goal:** [What this project delivers]
**Depends on:** [prior project name]

**Refines:**
- [ ] 01: [Brief description]

## 📋 Planned Milestone: v2.0 [Name]

**Goal:** [What v2.0 delivers]
**Prerequisite:** v1.1 complete
**Estimated projects:** [N]

[Project outlines without detailed refines — detail added when milestone begins]

| Project | Focus | Research |
|---------|-------|----------|
| [name] | [Focus] | Unlikely |
| [name] | [Focus] | Likely |
| [name] | [Focus] | Unlikely |
| [name] | [Focus] | Unlikely |

## ✅ Completed Milestones

<details>
<summary>v1.0 MVP — Shipped YYYY-MM-DD</summary>

### [name]
**Goal:** [What was delivered]
**Refines:** 3 complete

- [x] 01: [Description]
- [x] 02: [Description]
- [x] 03: [Description]

### [name]
**Goal:** [What was delivered]
**Refines:** 2 complete

- [x] 01: [Description]
- [x] 02: [Description]

### [name]
**Goal:** [What was delivered]
**Refines:** 1 complete

- [x] 01: [Description]

### [name]
**Goal:** [What was delivered]
**Refines:** 1 complete

- [x] 01: [Description]

**Milestone archive:** See [milestones/v1.0-mvp.md](milestones/v1.0-mvp.md)

</details>

---
*Last updated: [YYYY-MM-DD]*
```

---

## Section Specifications

### Overview
**Purpose:** High-level journey description.
**Length:** One paragraph.
**Update:** When project direction changes significantly.

### Current Milestone
**Purpose:** Quick reference to active milestone.
**Contains:** Name, version, status, project progress.
**Update:** After each project completion.

### Projects Table
**Purpose:** At-a-glance view of all projects.
**Contains:** Project name, refine count, status, completion date. Projects are ordered by list position.
**Update:** After each refine/project completion.

### Project Details
**Purpose:** Detailed breakdown of each project.
**Contains:**
- **Goal:** Specific deliverable outcome
- **Depends on:** Project dependencies with reason
- **Research:** Likely/Unlikely with justification
- **Scope:** Bullet list of deliverables
- **Refines:** Checklist with brief descriptions

**Update:** During planning; status after completion.

---

## Project Ordering

Projects execute in list order — top to bottom. To insert an urgent project, add it between the relevant existing entries and note the reason.

---

## Depth Configuration

Project count depends on project depth:

| Depth | Typical Projects | Refines/Project |
|-------|------------------|---------------|
| Quick | 3-5 | 1-3 |
| Standard | 5-8 | 3-5 |
| Comprehensive | 8-12 | 5-10 |

**Key principle:** Derive projects from actual work. Depth determines compression tolerance, not a target to hit.

---

## Research Flags

- **Unlikely:** Internal patterns, CRUD operations, established conventions
- **Likely:** External APIs, new libraries, architectural decisions

When `Research: Likely`:
- Include `Research topics:` field
- Consider research phase before implementation
- Discovery may be required during planning

---

## Status Values

| Status | Meaning |
|--------|---------|
| Not started | Phase hasn't begun |
| In progress | Actively working |
| Complete | All refines done (add date) |
| Deferred | Pushed to later (add reason) |

---

## Milestone Sections

### Active Milestone
**Purpose:** Currently executing milestone with full project details.
**Contains:**
- Goal and status summary
- Progress bar visualization
- All projects with refine checklists
- Research flags where applicable

### Planned Milestone
**Purpose:** Defined but not started milestone.
**Contains:**
- Goal and prerequisites
- Project table (focus areas, research likelihood)
- NOT detailed refines (added when milestone begins)

### Completed Milestones
**Purpose:** Historical record in collapsible sections.
**Contains:**
- Summary header with ship date
- Project details (collapsed by default)
- Link to full milestone archive

**Key principle:** Completed work should not consume visual space — collapse it.
