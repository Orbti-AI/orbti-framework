---
name: orbit:integrate
description: Reconcile plan vs actual and close the loop
argument-hint: "[plan-path]"
allowed-tools: [Read, Write, Bash, Glob, Grep, AskUserQuestion, Task]
---

<model>sonnet</model>

<objective>
Reconcile plan versus actual results, create INTEGRATE.md, and close the loop.

**When to use:** After BUILD phase complete. This is MANDATORY - never skip INTEGRATE.

Creates INTEGRATE.md documenting what was built, decisions made, and any deferred issues.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/integrate.md
@~/.claude/orbit-framework/templates/INTEGRATE.md
</execution_context>

<context>
Plan path: $ARGUMENTS

@.orbit/STATE.md
@{plan-path} (the LOOP.md being unified)
</context>

<process>

<step name="validate_preconditions">
1. Confirm LOOP.md exists at $ARGUMENTS path
2. Confirm BUILD phase was executed (tasks completed)
3. If INTEGRATE.md already exists: "Loop already closed. SUMMARY: {path}"
</step>

<step name="reconcile">
Follow workflow: @~/.claude/orbit-framework/workflows/integrate.md

Compare plan to actual:
- Which tasks completed as planned?
- Any deviations from plan?
- Decisions made during execution?
- Issues discovered but deferred?
</step>

<step name="create_summary">
Create INTEGRATE.md in same directory as LOOP.md:
- Document what was built
- Record acceptance criteria results
- Note any deferred issues
- Capture decisions made
- List files created/modified
</step>

<step name="update_state">
Update STATE.md:
- Loop position: REFINE ✓ → BUILD ✓ → INTEGRATE ✓
- Project progress if plan completes project
- Performance metrics (duration)
- Session continuity (next action)
</step>

<step name="report">
Display:
```
Loop Closed
════════════════════════════════════════

Plan: {plan-path}
Summary: {summary-path}

REFINE ──▶ BUILD ──▶ INTEGRATE
  ✓        ✓        ✓

Next: [project complete message or next plan]

════════════════════════════════════════
```
</step>

</process>

<success_criteria>
- [ ] INTEGRATE.md created
- [ ] STATE.md updated with loop closure
- [ ] User knows next action
</success_criteria>
