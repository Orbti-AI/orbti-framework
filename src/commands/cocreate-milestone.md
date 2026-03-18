---
name: orbit:cocreate-milestone
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
@~/.claude/orbit-framework/workflows/cocreate-milestone.md
</execution_context>

<context>
@.orbit/PROJECT.md
@.orbit/STATE.md
@.orbit/ROADMAP.md
@.orbit/MILESTONES.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/cocreate-milestone.md
</process>

<success_criteria>
- [ ] MILESTONE-CONTEXT.md created with vision
- [ ] Key themes and goals articulated
- [ ] Ready for /orbit:milestone command
</success_criteria>
