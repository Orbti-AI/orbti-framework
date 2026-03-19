---
name: orbti:status
description: "[DEPRECATED] Use /orbti:progress instead"
argument-hint:
allowed-tools: [Read]
---

<model>haiku</model>

> **⚠️ DEPRECATED:** This command is deprecated. Use `/orbti:progress` instead.
>
> `/orbti:progress` provides the same information plus:
> - Visual milestone progress
> - Smarter routing with single next-action suggestion
> - Optional context argument for tailored suggestions

<objective>
Display current loop position (REFINE/BUILD/INTEGRATE) and project progress.

**When to use:** Use `/orbti:progress` instead for better routing.
</objective>

<execution_context>
</execution_context>

<context>
@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>

<step name="read_state">
Read STATE.md and extract:
- Current milestone
- Current project (X of Y)
- Current refine status
- Loop position (REFINE/BUILD/INTEGRATE)
- Last activity
</step>

<step name="display_status">
Display formatted status:

```
ORBTI Status
════════════════════════════════════════

Milestone: [name]
Project: [X of Y] ([project name])
Refine: [status]

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
- If REFINE needed: "Run /orbti:refine to create refine"
- If REFINE ready: "Approve refine, then run /orbti:build"
- If BUILD complete: "Run /orbti:integrate to close loop"
- If INTEGRATE complete: "Loop closed. Ready for next project."
</step>

</process>

<success_criteria>
- [ ] Loop position displayed clearly
- [ ] Project progress shown
- [ ] Next action suggested
</success_criteria>
