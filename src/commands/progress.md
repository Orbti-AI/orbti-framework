---
name: orbti:progress
description: Smart status with routing - suggests ONE next action
argument-hint: "[context]"
allowed-tools: [Read]
---

<objective>
Show current progress and **route to exactly ONE next action**. Prevents decision fatigue by suggesting a single best path.

**When to use:**
- Mid-session check on progress
- After `/orbti:resume` for more context
- When unsure what to do next
- To get a tailored suggestion based on your current focus
</objective>

<model>haiku</model>

<execution_context>
</execution_context>

<context>
$ARGUMENTS

@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>

<step name="load_state">
Read `.orbti/STATE.md` and `.orbti/ROADMAP.md`:
- Current phase and total phases
- Current refine (if any)
- Loop position (REFINE/BUILD/INTEGRATE markers)
- Roadmap progress
- Performance metrics (if tracked)
- Blockers or concerns
</step>

<step name="calculate_progress">
Determine overall progress:

**Milestone Progress:**
- Phases complete: X of Y
- Current phase progress: Z%

**Current Loop:**
- Position: REFINE/BUILD/INTEGRATE
- Status: [what's happening]
</step>

<step name="consider_user_context">
**If `[context]` argument provided:**

User has given additional context about their current focus or constraint.
Factor this into routing decision:
- "I need to fix a bug first" → prioritize that over planned work
- "I only have 30 minutes" → suggest smaller scope
- "I want to finish this phase" → stay on current path
- "I'm stuck on X" → suggest debug or research approach

**If no argument:** Use default routing based on state alone.
</step>

<step name="determine_routing">
Based on state (+ user context if provided), determine **ONE** next action:

**Default routing (no user context):**

| Situation | Single Suggestion |
|-----------|-------------------|
| No refine exists | `/orbti:refine` |
| Refine awaiting approval | "Approve refine to proceed" |
| Refine approved, not executed | `/orbti:build [path]` |
| Applied, not unified | `/orbti:integrate [path]` |
| Loop complete, more phases | `/orbti:refine` (next phase) |
| Milestone complete | "Create next milestone or ship" |
| Blockers present | "Address blocker: [specific]" |
| Context at DEEP/CRITICAL | `/orbti:pause` |

**With user context:** Adjust suggestion to align with stated intent.

**IMPORTANT:** Suggest exactly ONE action. Not multiple options.
</step>

<step name="display_progress">
Show progress with single routing:

```
════════════════════════════════════════
ORBTI PROGRESS
════════════════════════════════════════

Milestone: [name] - [X]% complete
├── Phase 1: [name] ████████████ Done
├── Phase 2: [name] ████████░░░░ 70%
├── Phase 3: [name] ░░░░░░░░░░░░ Pending
└── Phase 4: [name] ░░░░░░░░░░░░ Pending

Current Loop: Phase 2, Refine 02-03
┌─────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE          │
│    ✓        ✓        ○             │
└─────────────────────────────────────┘

────────────────────────────────────────
▶ NEXT: /orbti:integrate .orbti/projects/02-features/02-03-REFINE.md
  Close the loop and update state.
────────────────────────────────────────

Type "yes" to proceed, or provide context for a different suggestion.
```
</step>

<step name="context_advisory">
If context is at DEEP or CRITICAL bracket:

```
⚠️ Context Advisory: Session at [X]% capacity.
   Recommended: /orbti:pause before continuing.
```
</step>

</process>

<success_criteria>
- [ ] Overall progress displayed visually
- [ ] Current loop position shown
- [ ] Exactly ONE next action suggested (not multiple)
- [ ] User context considered if provided
- [ ] Context advisory shown if needed
</success_criteria>
