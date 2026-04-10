---
name: orbti:observe
description: Explore and articulate what you want to build before planning
argument-hint: "[topic]"
allowed-tools: [Read, Write, AskUserQuestion]
---

<model>opus</model>

<objective>
Facilitate vision discussion before a project exists and create context handoff for /orbti:refine.

**When to use:** Before planning, when goals and approach need exploration. Runs before /orbti:refine — no project exists yet.
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/observe.md
</execution_context>

<context>
Topic: $ARGUMENTS (optional)

@.orbti/PROJECT.md
@.orbti/STATE.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/observe.md
</process>

<success_criteria>
- [ ] CONTEXT.md created in project directory
- [ ] Goals and approach articulated
- [ ] Ready for /orbti:refine command
</success_criteria>
