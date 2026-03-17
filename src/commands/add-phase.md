---
name: orbit:add-phase
description: Add a new project to current milestone
argument-hint: "[project-name]"
allowed-tools: [Read, Write, Edit, Bash]
---

<objective>
Add a new project to the current milestone's roadmap.

**When to use:** Scope expansion during milestone, adding planned work.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/roadmap-management.md
</execution_context>

<context>
$ARGUMENTS

@.orbit/PROJECT.md
@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/roadmap-management.md

Execute: **add-project** operation
</process>

<success_criteria>
- [ ] Project added to ROADMAP.md
- [ ] Project directory created
- [ ] STATE.md updated with new project
</success_criteria>
