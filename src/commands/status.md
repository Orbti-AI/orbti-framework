---
name: orbit:status
description: "[DEPRECATED] Use /orbit:progress instead"
argument-hint:
allowed-tools: [Read]
---

> **⚠️ DEPRECATED:** This command is deprecated. Use `/orbit:progress` instead.
>
> `/orbit:progress` provides the same information plus:
> - Visual milestone progress
> - Smarter routing with single next-action suggestion
> - Optional context argument for tailored suggestions

<objective>
Display current loop position (REFINE/BUILD/INTEGRATE) and phase progress.

**When to use:** Use `/orbit:progress` instead for better routing.
</objective>

<execution_context>
</execution_context>

<context>
@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>

<step name="read_state">
Read STATE.md and extract:
- Current milestone
- Current phase (X of Y)
- Current plan status
- Loop position (REFINE/BUILD/INTEGRATE)
- Last activity
</step>

<step name="display_status">
Display formatted status:

```
ORBIT Status
════════════════════════════════════════

Milestone: [name]
Phase: [X of Y] ([phase name])
Plan: [status]

Loop Position:
┌─────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE          │
│   [✓/○]    [✓/○]    [✓/○]          │
└─────────────────────────────────────┘

Last: [timestamp] — [activity]
Next: [recommended action]

════════════════════════════════════════
```
</step>

<step name="suggest_next">
Based on loop position, suggest next action:
- If REFINE needed: "Run /orbit:refine to create plan"
- If REFINE ready: "Approve plan, then run /orbit:build"
- If BUILD complete: "Run /orbit:integrate to close loop"
- If INTEGRATE complete: "Loop closed. Ready for next phase."
</step>

</process>

<success_criteria>
- [ ] Loop position displayed clearly
- [ ] Phase progress shown
- [ ] Next action suggested
</success_criteria>
