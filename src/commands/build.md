---
name: orbit:build
description: Execute an approved REFINE plan
argument-hint: "[plan-path] [--background]"
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, Task]
---

<objective>
Execute an approved LOOP.md file, running tasks in order with verification at each step.

**When to use:** After REFINE phase complete and plan is approved.

**Flags:**
- `--background` — run autonomously in background (requires `autonomous: true` plan)
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/build.md
@~/.claude/orbit-framework/references/checkpoints.md
</execution_context>

<context>
Plan path: $ARGUMENTS

@.orbit/STATE.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/build.md
</process>

<success_criteria>
- [ ] All tasks executed and verified
- [ ] Checkpoints handled (or background agent notified on completion)
- [ ] STATE.md updated with BUILD complete
- [ ] User knows next action: /orbit:integrate
</success_criteria>
