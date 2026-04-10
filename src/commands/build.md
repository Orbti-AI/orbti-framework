---
name: orbti:build
description: Execute an approved REFINE refine
argument-hint: "[refine-path]"
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, Task]
---

<objective>
Execute an approved REFINE.md file, running tasks in order with verification at each step.

**When to use:** After REFINE phase complete and refine is approved.

</objective>

<model>sonnet</model>

<execution_context>
@./.claude/orbti-framework/workflows/build.md
@./.claude/orbti-framework/references/checkpoints.md
@./.claude/orbti-framework/references/model-routing.md
</execution_context>

<context>
Refine path: $ARGUMENTS

@.orbti/STATE.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/build.md
</process>

<success_criteria>
- [ ] All tasks executed and verified
- [ ] Checkpoints handled (or background agent notified on completion)
- [ ] STATE.md updated with BUILD complete
- [ ] User knows next action: /orbti:integrate
</success_criteria>
