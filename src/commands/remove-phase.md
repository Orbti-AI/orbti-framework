---
name: orbit:remove-phase
description: Remove a future (not started) project
argument-hint: "<project-number-or-name>"
allowed-tools: [Read, Write, Edit, Bash]
---

<model>haiku</model>

<objective>
Remove a future project from the roadmap and clean up its directory.

**When to use:** Scope reduction, removing projects that haven't started.
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

Execute: **remove-project** operation
</process>

<success_criteria>
- [ ] Project removed from ROADMAP.md
- [ ] Project directory cleaned up (if empty)
- [ ] Subsequent projects renumbered
- [ ] STATE.md updated
</success_criteria>
