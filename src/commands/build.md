---
name: orbit:build
description: Execute an approved REFINE plan
argument-hint: "[plan-path] [--background]"
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion]
---

<objective>
Execute an approved LOOP.md file, handling checkpoints as they occur.

**When to use:** After REFINE phase complete and plan is approved.

Executes tasks in sequence, pauses at checkpoints for user input, reports completion.

**Flags:**
- `--background` — run autonomously in the background (only works for plans with `autonomous: true`; plans with checkpoints must run in foreground)
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

<step name="validate_plan">
1. Confirm plan file exists at $ARGUMENTS path
2. Error if not found: "Plan not found: {path}"
3. Derive SUMMARY path (replace LOOP.md with INTEGRATE.md)
4. If SUMMARY exists: "Plan already executed. SUMMARY: {path}"
   - Offer: re-execute or exit
</step>

<step name="execute">
Follow workflow: @~/.claude/orbit-framework/workflows/build.md

Execute tasks sequentially. For each task:
- Read task definition
- Execute action
- Run verification
- Confirm done criteria
</step>

<step name="handle_checkpoints">
When a checkpoint task is reached:

**checkpoint:decision**
- Present decision context and options
- Wait for user selection
- Record decision
- Continue execution

**checkpoint:human-verify**
- Present what was built
- Present verification steps
- Wait for "approved" or issue description
- If issues: address and re-verify
- Continue execution

**checkpoint:human-action**
- Present required action
- Wait for "done" confirmation
- Continue execution
</step>

<step name="complete">
After all tasks complete:
- Report: "BUILD complete. Run /orbit:integrate to close loop."
- Show files modified
- Show SUMMARY path to create
</step>

</process>

<success_criteria>
- [ ] All tasks executed
- [ ] All checkpoints handled
- [ ] User informed of completion
- [ ] Next action clear (run /orbit:integrate)
</success_criteria>
