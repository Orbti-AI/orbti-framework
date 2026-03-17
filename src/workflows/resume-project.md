<purpose>
Resume ORBIT work after a session break. Reads STATE.md to restore context, determines current loop position, and routes to exactly ONE next action. Includes handoff lifecycle management.
</purpose>

<when_to_use>
- Starting a new session on an existing ORBIT project
- Context was cleared (new conversation)
- Handoff from another session
- User asks to "continue" or "resume" ORBIT work
</when_to_use>

<loop_context>
Determined dynamically by reading STATE.md.
This workflow figures out where we are, not assumes it.
</loop_context>

<philosophy>
**Single next action:** Resume determines state and suggests exactly ONE action.
No multiple options. Prevents decision fatigue. User can redirect if needed.

**Handoff lifecycle:** Handoffs are consumed on resume, then archived or deleted.
</philosophy>

<required_reading>
@.orbit/STATE.md
</required_reading>

<references>
@~/.claude/orbit-framework/references/context-management.md
@~/.claude/orbit-framework/references/loop-phases.md
</references>

<process>

<step name="verify_orbit_exists" priority="first">
1. Check for .orbit/ directory:
   ```bash
   ls .orbit/STATE.md 2>/dev/null
   ```
2. If not found:
   - "No ORBIT project found. Run /orbit:init first."
   - Exit workflow
3. If found: proceed with resume
</step>

<step name="detect_handoffs">
**Check for handoff files:**

1. Search for handoff files:
   ```bash
   ls -t .orbit/HANDOFF*.md 2>/dev/null | head -5
   ```

2. If handoff argument provided ($ARGUMENTS):
   - Use specified handoff path
   - Validate it exists

3. If no argument but handoffs found:
   - Use most recent handoff (by modification time)
   - Note: `.orbit/HANDOFF-{context}.md` is standard pattern

4. Track handoff for lifecycle:
   - Store path for later archive/delete
   - Will be consumed after resume proceeds
</step>

<step name="load_state">
1. Read `.orbit/STATE.md`
2. Extract:
   - Current Position (phase, plan, status)
   - Loop Position (REFINE/BUILD/INTEGRATE markers)
   - Last activity (what was happening)
   - Session Continuity section:
     - Stopped at
     - Next action
     - Resume file
     - Resume context
</step>

<step name="load_resume_context">
1. If handoff detected in previous step:
   - Read handoff file content
   - Present as "HANDOFF CONTEXT DETECTED" section
   - Extract key information (decisions, gaps, next actions)

2. If resume file specified in STATE.md (and no handoff):
   - Read the resume file (REFINE, SUMMARY)

3. Build mental picture of:
   - What was accomplished
   - What's in progress
   - What's next
   - Any decisions or gaps from handoff
</step>

<step name="determine_single_action">
Based on loop position, determine **exactly ONE** next action:

| Loop State | Single Next Action |
|------------|-------------------|
| REFINE ○ (no plan yet) | `/orbit:refine` |
| REFINE ✓, BUILD ○ (plan awaiting approval) | `/orbit:build [plan-path]` |
| REFINE ✓, BUILD ✓, INTEGRATE ○ (executed, not reconciled) | `/orbit:integrate [plan-path]` |
| All ✓ (loop complete) | `/orbit:refine` (next phase) |
| Blocked | "Address blocker: [specific issue]" |

**Do NOT offer multiple options.** Pick the ONE correct action.
</step>

<step name="report_and_route">
Display to user with ONE next action:

```
════════════════════════════════════════
ORBIT PROJECT RESUMED
════════════════════════════════════════

Project: [from PROJECT.md]
Phase: [N] of [M] - [Phase Name]
Plan: [NN-PP] - [plan description]

Loop Position:
┌─────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE          │
│   [✓/○]    [✓/○]    [✓/○]          │
└─────────────────────────────────────┘

Last Session: [timestamp]
Stopped at: [what was happening]

────────────────────────────────────────
▶ NEXT: [single command with path]
  [brief description of what it does]
────────────────────────────────────────

Type "yes" to proceed, or provide context for a different action.
```

**IMPORTANT:** Do NOT show numbered options (1, 2, 3, 4).
Show exactly ONE suggested action with the standard ORBIT routing format.
</step>

</process>

<step name="handoff_lifecycle">
**After user proceeds with work:**

When user confirms next action (e.g., "yes", "1", "approved"):

1. **Archive handoff** (if one was consumed):
   ```bash
   mkdir -p .orbit/handoffs/archive
   mv .orbit/HANDOFF-{context}.md .orbit/handoffs/archive/
   ```
   - Preserves history while removing from active path
   - Alternative: delete if archive not needed

2. **Clean orphaned handoffs:**
   ```bash
   # Find handoffs older than current phase
   find .orbit -maxdepth 1 -name "HANDOFF*.md" -mtime +7 -type f
   ```
   - Move to archive or delete
   - Prevents accumulation of stale handoffs

3. **Update STATE.md:**
   - Clear "Resume file" if it pointed to handoff
   - Handoff context now integrated into session
</step>

</process>

<output>
- Context restored from STATE.md and/or handoff
- Handoff content presented if detected
- User informed of current position
- Exactly ONE next action suggested
- User can proceed or redirect
- Handoff archived after work proceeds
</output>

<error_handling>
**STATE.md corrupted or incomplete:**
- Report what's missing
- Suggest: `/orbit:init` to reinitialize (destructive) or manual repair

**Conflicting information:**
- STATE.md says X, but files suggest Y
- Report discrepancy
- Ask user to clarify actual state

**No resume context:**
- If SESSION CONTINUITY section empty:
- Fall back to loop position
- Suggest based on what files exist

**Stale handoff:**
- If handoff older than STATE.md modifications
- Trust STATE.md, note handoff may be outdated
</error_handling>
