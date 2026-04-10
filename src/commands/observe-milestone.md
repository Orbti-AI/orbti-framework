---
name: orbti:observe-milestone
description: Explore and articulate next milestone vision
argument-hint: ""
allowed-tools: [Read, Write, AskUserQuestion]
---

<model>opus</model>

<objective>
Facilitate vision discussion for the next milestone and create context handoff.

**When to use:** Before creating a new milestone, when scope needs exploration.
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/observe-milestone.md
</execution_context>

<context>
@.orbti/PROJECT.md
@.orbti/STATE.md
@.orbti/ROADMAP.md
@.orbti/MILESTONES.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/observe-milestone.md
</process>

<success_criteria>
- [ ] MILESTONE-CONTEXT.md created with vision
- [ ] Key themes and goals articulated
- [ ] Ready for /orbti:milestone command
</success_criteria>
