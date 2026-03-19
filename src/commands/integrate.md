---
name: orbti:integrate
description: Reconcile refine vs actual and close the loop
argument-hint: "[refine-path]"
allowed-tools: [Read, Write, Bash, Glob, Grep, AskUserQuestion, Task]
---

<model>sonnet</model>

<objective>
Reconcile refine versus actual results, create INTEGRATE.md, and close the loop.

**When to use:** After BUILD phase complete. This is MANDATORY - never skip INTEGRATE.

Creates INTEGRATE.md documenting what was built, decisions made, and any deferred issues.
</objective>

<execution_context>
@~/.claude/orbti-framework/workflows/integrate.md
@~/.claude/orbti-framework/templates/INTEGRATE.md
</execution_context>

<context>
Refine path: $ARGUMENTS

@.orbti/STATE.md
@{refine-path} (the REFINE.md being unified)
</context>

<process>

<step name="validate_preconditions">
1. Confirm REFINE.md exists at $ARGUMENTS path
2. Confirm BUILD phase was executed (tasks completed)
3. If INTEGRATE.md already exists: "Loop already closed. SUMMARY: {path}"
</step>

<step name="reconcile">
Follow workflow: @~/.claude/orbti-framework/workflows/integrate.md

Compare refine to actual:
- Which tasks completed as planned?
- Any deviations from refine?
- Decisions made during execution?
- Issues discovered but deferred?
</step>

<step name="create_summary">
Create INTEGRATE.md in same directory as REFINE.md:
- Document what was built
- Record acceptance criteria results
- Note any deferred issues
- Capture decisions made
- List files created/modified
</step>

<step name="update_state">
Update STATE.md:
- Loop position: REFINE ✓ → BUILD ✓ → INTEGRATE ✓
- Project progress if refine completes project
- Performance metrics (duration)
- Session continuity (next action)
</step>

<step name="report">
Display:
```
Loop Closed
════════════════════════════════════════

Refine: {refine-path}
Summary: {summary-path}

REFINE ──▶ BUILD ──▶ INTEGRATE
  ✓        ✓        ✓

Next: [project complete message or next refine]

════════════════════════════════════════
```
</step>

</process>

<success_criteria>
- [ ] INTEGRATE.md created
- [ ] STATE.md updated with loop closure
- [ ] User knows next action
</success_criteria>
