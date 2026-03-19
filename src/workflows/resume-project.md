<purpose>
Resume ORBTI work after a session break. Reads STATE.md to restore context, determines current loop position, and routes to exactly ONE next action. Includes handoff lifecycle management.
</purpose>

<when_to_use>
- Starting a new session on an existing ORBTI project
- Context was cleared (new conversation)
- Handoff from another session
- User asks to "continue" or "resume" ORBTI work
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
@.orbti/STATE.md
</required_reading>

<references>
@~/.claude/orbti-framework/references/context-management.md
@~/.claude/orbti-framework/references/loop-phases.md
</references>

<process>

<step name="verify_orbti_exists" priority="first">
1. Check for .orbti/ directory:
   ```bash
   ls .orbti/STATE.md 2>/dev/null
   ```
2. If not found:
   - "No ORBTI project found. Run /orbti:init first."
   - Exit workflow
3. If found: proceed with resume
</step>

<step name="detect_handoffs">
**Check for handoff files:**

1. Identify active/paused projects from STATE.md Overview
2. For each active/paused project, check for handoff in its folder:
   ```bash
   ls -t .orbti/projects/{project}/HANDOFF*.md 2>/dev/null | head -1
   ```
3. If handoff argument provided ($ARGUMENTS — e.g. "01" or "auth"):
   - Resolve to project folder, look for handoff there
4. Track handoff path for lifecycle (will be archived after resume)
</step>

<step name="load_state">
1. Read `.orbti/STATE.md`
2. Extract:
   - Current Position (project, refine, status)
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

<step name="detect_active_projects">
**Check how many projects are In Progress or Paused:**

From the Projects Overview in STATE.md, count rows with status `🔵 In Progress` or `⏸ Paused`.

**If exactly one active project:** proceed to `determine_single_action` directly.

**If multiple active/paused projects:**
```
════════════════════════════════════════
MULTIPLE PROJECTS IN PROGRESS
════════════════════════════════════════

Projects Overview:
┌────────────────────────────────────────────────────────────┐
│  #   Project              Loops   Status          Position  │
│  01  [name]               1/3     ⏸ Paused        ✓ ◉ ○    │
│  02  [name]               0/2     🔵 In Progress  ◉ ○ ○    │
└────────────────────────────────────────────────────────────┘

Which project do you want to continue?
Type the project number or name (e.g., "01" or "auth").
════════════════════════════════════════
```

Wait for user selection. Once selected:
- Set chosen project as active (`🔵 In Progress`)
- Set others back to `⏸ Paused` in the Overview
- Load `.orbti/projects/{project}/HANDOFF-*.md` if it exists
- Proceed to `determine_single_action` for the chosen project
</step>

<step name="determine_single_action">
Based on the active project's loop position, determine **exactly ONE** next action:

| Loop State | Single Next Action |
|------------|-------------------|
| REFINE ○ (no refine yet) | `/orbti:refine` |
| REFINE ✓, BUILD ○ (refine awaiting approval) | `/orbti:build [refine-path]` |
| REFINE ✓, BUILD ✓, INTEGRATE ○ (executed, not reconciled) | `/orbti:integrate [refine-path]` |
| All ✓ (loop complete) | `/orbti:refine` (next loop) |
| Blocked | "Address blocker: [specific issue]" |

**Do NOT offer multiple options.** Pick the ONE correct action.
</step>

<step name="report_and_route">
Display the Projects Overview, then the next action for the active project:

```
════════════════════════════════════════
ORBTI RESUMED
════════════════════════════════════════

Projects Overview:
┌────────────────────────────────────────────────────────────┐
│  #   Project              Loops   Status          Position  │
│  01  [name]               1/3     ⏸ Paused        ✓ ◉ ○    │
│  02  [name]               0/2     🔵 In Progress  ◉ ○ ○    │
│  03  [name]               0/2     ○ Pending       ○ ○ ○    │
│  04  [name]               3/3     ✅ Complete      ✓ ✓ ✓    │
└────────────────────────────────────────────────────────────┘
Milestone: [X] of [Y] projects complete

────────────────────────────────────────
Active: Project [N] — [Name]
Stopped at: [what was happening]
────────────────────────────────────────
▶ NEXT: [single command with path]
  [brief description of what it does]
────────────────────────────────────────

Type "yes" to proceed, or type a project number/name to switch projects.
```

**IMPORTANT:** If only one active project — ONE next action, no choice needed.
If multiple active/paused — run `detect_active_projects` step first to let user pick.
</step>

</process>

<step name="handoff_lifecycle">
**After user proceeds with work:**

When user confirms next action (e.g., "yes", "1", "approved"):

1. **Archive handoff** (if one was consumed):
   ```bash
   mkdir -p .orbti/handoffs/archive
   mv .orbti/HANDOFF-{context}.md .orbti/handoffs/archive/
   ```
   - Preserves history while removing from active path
   - Alternative: delete if archive not needed

2. **Clean orphaned handoffs:**
   ```bash
   # Find handoffs older than current phase
   find .orbti -maxdepth 1 -name "HANDOFF*.md" -mtime +7 -type f
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
- Suggest: `/orbti:init` to reinitialize (destructive) or manual repair

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
