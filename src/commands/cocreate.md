---
name: orbti:cocreate
description: Explore and articulate project vision before planning
argument-hint: "<project-number>"
allowed-tools: [Read, Write, AskUserQuestion]
---

<model>opus</model>

<objective>
Facilitate vision discussion for a specific project and create context handoff.

**When to use:** Before planning a project, when goals and approach need exploration.
</objective>

<execution_context>
@~/.claude/orbti-framework/workflows/cocreate.md
</execution_context>

<context>
Project number: $ARGUMENTS (required)

@.orbti/PROJECT.md
@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbti-framework/workflows/cocreate.md
</process>

<success_criteria>
- [ ] CONTEXT.md created in project directory
- [ ] Goals and approach articulated
- [ ] Ready for /orbti:refine command
</success_criteria>
