<specialized_workflow_integration>

## Purpose

SKILLS.md enables explicit tracking of specialized skills, commands, and workflows within ORBTI governance. Instead of ad-hoc skill invocation, projects declare which skills apply, when they're required, and INTEGRATE verifies they were actually used.

**Core principle:** If a skill is important enough to use, it's important enough to track.

## Why This Matters

### The Problem
Users build specialized skills for their work (e.g., `/revops-expert` for persuasion copy, `/frontend-design` for UI components). Without explicit integration:
- Skills are invoked ad-hoc, inconsistently
- No verification that required skills were used
- No gap analysis for continuous improvement
- Knowledge about "which skills for which work" lives only in human memory

### The Solution
Layered reinforcement through the ORBTI loop:

```
SKILLS.md          →  "This project uses X, Y, Z skills"
       ↓
ROADMAP.md         →  "Project 2 requires X skill"
       ↓
REFINE.md              →  "Task 2.1 invokes /frontend-design"
       ↓
INTEGRATE          →  "Was skill invoked? Gap noted if not"
```

## Skill Flow

### 1. Declaration (SKILLS.md)
At project setup, declare which skills apply:

```markdown
| Work Type | Skill/Command | Priority | When Required |
|-----------|---------------|----------|---------------|
| Persuasion copy | /revops-expert | required | Before headlines, CTAs |
| UI components | /frontend-design | optional | Before writing HTML |
```

### 2. Project Annotation (ROADMAP.md)
Optionally annotate projects with required skills:

```markdown
### Project 4: Landing Page
**Required Skills:** /revops-expert (copy), /frontend-design (UI)
```

### 3. Task Invocation (REFINE.md)
Refines can reference skills in pre-implementation or task notes:

```markdown
## Pre-Implementation
- [ ] Load /revops-expert (Hormozi frameworks)

## Tasks
1. Write headline copy
   - **Skill:** /revops-expert (value equation)
```

### 4. Verification (INTEGRATE)
During INTEGRATE, audit skill invocation:

| Expected | Invoked | Gap? | Notes |
|----------|---------|------|-------|
| /revops-expert | ✓ | - | Loaded Hormozi stack |
| /frontend-design | ○ | Yes | Used inline CSS instead |

Gaps are documented in STATE.md Deviations, not blocked.

## Priority Levels

### Required
- **Definition:** Skill should be invoked for this type of work
- **Verification:** Gap documented in STATE.md if not used
- **Use case:** Core skills that significantly impact quality

### Optional
- **Definition:** Skill is available but not mandatory
- **Verification:** Informational only, no gap logged
- **Use case:** Nice-to-have skills, situational enhancements

## Verification Behavior

### Manual During INTEGRATE (Not Hook-Based)
Skill usage is verified manually during INTEGRATE phase:
1. Read SKILLS.md for declared skills
2. Review conversation/session for invocations
3. Mark each required skill as invoked or gap
4. Document gaps in STATE.md Deviations

**Why not automatic?** Hook-based tracking is complex, may miss invocations, and adds fragility. Manual verification during INTEGRATE keeps it simple and reliable.

### Warn, Don't Block
Gaps generate warnings, not blocking errors:
- Gaps are documented for learning
- User can note "intentional deviation"
- INTEGRATE still completes
- Continuous improvement, not rigid enforcement

## Examples

### Example 1: Client Website Project

```markdown
# Skills

## Project-Level Dependencies

| Work Type | Skill/Command | Priority | When Required |
|-----------|---------------|----------|---------------|
| Persuasion copy | /revops-expert | required | Headlines, CTAs, offers |
| UI components | /frontend-design | required | All HTML generation |
| Pricing sections | /revops-expert:hormozi | optional | Offer stacking phases |

## Phase Overrides

| Phase | Additional Skills | Notes |
|-------|-------------------|-------|
| 3 (Pricing) | /revops-expert:hormozi | Value stacking for offers |

## Templates & Assets

| Asset Type | Location | When Used |
|------------|----------|-----------|
| Hero template | templates/hero-section.html | Homepage hero |
| Testimonial | templates/testimonial-card.html | Social proof sections |
```

### Example 2: Technical Project

```markdown
# Skills

## Project-Level Dependencies

| Work Type | Skill/Command | Priority | When Required |
|-----------|---------------|----------|---------------|
| API documentation | /docs-generator | optional | After API changes |
| Database changes | /migration-helper | required | Schema modifications |
```

### Example 3: Product with UI Layer

Solution Intent (WHO/WHAT/FEEL) and agent learning (RUNBOOK) are built into the ORBTI loop — no declaration needed. Declare agentic-design as required skill to enforce visual craft:

```markdown
# Skills

## Project-Level Dependencies

| Work Type | Skill/Command | Priority | When Required |
|-----------|---------------|----------|---------------|
| UI screens | /agentic-design:init | required | Before first UI task in BUILD |
| E2E tests | playwright | required | After UI feature built |
```

## Evolution and Improvement

### Pattern Recognition
Over time, SKILLS.md reveals patterns:
- Which skills are consistently used
- Which are often skipped (maybe not required?)
- New skills that should be added

### Updating Patterns
When intentional deviation becomes the norm:
1. Note deviation in INTEGRATE
2. Consider updating SKILLS.md
3. Log decision if significant change

## Integration Points

### With Init
During `/orbti:init`, optionally configure skills:
- "Do you have specialized skills for this project?"
- If yes, route to configuration workflow
- If no, skip (can add later via `/orbti:skills`)

### With INTEGRATE
During INTEGRATE, audit skill usage:
- Check SKILLS.md for declared skills
- Verify invocations in session
- Document gaps in STATE.md Deviations

### With PROJECT.md
Quick reference in PROJECT.md:

```markdown
## Skills

See: .orbti/SKILLS.md

Quick Reference:
- /revops-expert → Persuasion copy, offers, CTAs
- /frontend-design → UI components, HTML generation
```

</specialized_workflow_integration>
