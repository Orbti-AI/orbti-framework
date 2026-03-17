---
name: orbit:refine
description: Enter REFINE phase for current or new plan
argument-hint: "[phase-plan]"
allowed-tools: [Read, Write, Glob, AskUserQuestion]
---

<objective>
Create or continue a REFINE plan for the specified phase.

**When to use:** Starting new work or resuming incomplete plan.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/refine.md
@~/.claude/orbit-framework/templates/REFINE.md
@~/.claude/orbit-framework/references/plan-format.md
</execution_context>

<context>
$ARGUMENTS

@.orbit/PROJECT.md
@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/refine.md
</process>

<success_criteria>
- [ ] REFINE.md created in correct phase directory
- [ ] All acceptance criteria defined
- [ ] STATE.md updated with loop position
</success_criteria>
