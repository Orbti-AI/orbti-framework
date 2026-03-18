---
name: orbit:milestone
description: Create a new milestone in the project
argument-hint: "[milestone-name]"
allowed-tools: [Read, Write, Edit, Bash, Glob, AskUserQuestion]
---

<model>haiku</model>

<objective>
Create a new milestone with defined scope and phases.

**When to use:** Starting a new milestone cycle after completing the previous one.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/create-milestone.md
</execution_context>

<context>
$ARGUMENTS

@.orbit/PROJECT.md
@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/create-milestone.md
</process>

<success_criteria>
- [ ] Milestone created in MILESTONES.md
- [ ] ROADMAP.md updated with milestone grouping
- [ ] STATE.md reflects new milestone
</success_criteria>
