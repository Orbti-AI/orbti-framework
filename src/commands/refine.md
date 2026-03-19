---
name: orbti:refine
description: Enter REFINE phase for current or new refine
argument-hint: "[phase-refine]"
allowed-tools: [Read, Write, Glob, AskUserQuestion]
---

<objective>
Create or continue a REFINE refine for the specified phase.

**When to use:** Starting new work or resuming incomplete refine.
</objective>

<model>opus</model>

<execution_context>
@~/.claude/orbti-framework/workflows/refine.md
@~/.claude/orbti-framework/templates/REFINE.md
@~/.claude/orbti-framework/references/refine-format.md
@~/.claude/orbti-framework/references/model-routing.md
</execution_context>

<context>
$ARGUMENTS

@.orbti/PROJECT.md
@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbti-framework/workflows/refine.md
</process>

<success_criteria>
- [ ] REFINE.md created in correct project directory
- [ ] All acceptance criteria defined
- [ ] STATE.md updated with loop position
</success_criteria>
