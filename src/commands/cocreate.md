---
name: orbit:cocreate
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
@~/.claude/orbit-framework/workflows/cocreate.md
</execution_context>

<context>
Project number: $ARGUMENTS (required)

@.orbit/PROJECT.md
@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/cocreate.md
</process>

<success_criteria>
- [ ] CONTEXT.md created in project directory
- [ ] Goals and approach articulated
- [ ] Ready for /orbit:refine command
</success_criteria>
