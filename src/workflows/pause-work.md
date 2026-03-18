<purpose>
Create HANDOFF.md file and update STATE.md when pausing work mid-session. Enables seamless resumption in fresh session with full context restoration.
</purpose>

<when_to_use>
- Before ending a session (planned or context limit approaching)
- Switching to different project
- Context window at DEEP or CRITICAL bracket
- User explicitly pauses work
</when_to_use>

<loop_context>
Any position in REFINE/BUILD/INTEGRATE loop. Captures state regardless of where loop is.
</loop_context>

<required_reading>
@.orbit/STATE.md
@.orbit/PROJECT.md
</required_reading>

<references>
@~/.claude/orbit-framework/references/context-management.md
@~/.claude/orbit-framework/templates/HANDOFF.md
</references>

<process>

<step name="detect_position" priority="first">
1. Read `.orbit/STATE.md` to get:
   - Current project and plan
   - Loop position (REFINE/BUILD/INTEGRATE markers)
   - Last activity
2. Identify current working directory:
   ```bash
   ls -t .orbit/projects/*/LOOP*.md 2>/dev/null | head -1
   ```
</step>

<step name="gather_session_context">
**Collect complete state for handoff:**

Ask user (or infer from conversation):
1. **Work completed this session** - What got done?
2. **Work in progress** - What's partially done?
3. **Decisions made** - Key choices and rationale
4. **Blockers/issues** - Anything stuck?
5. **Mental context** - The approach, what you were thinking

If user doesn't provide, summarize from:
- Recent file modifications (`git status`)
- Conversation history
- STATE.md changes
</step>

<step name="create_handoff">
**Create HANDOFF file — named by project:**

```bash
# Generate filename scoped to current project
PROJECT=$(grep "Project:" .orbit/STATE.md | head -1 | sed 's/.*— //' | tr ' ' '-' | tr '[:upper:]' '[:lower:]')
TIMESTAMP=$(date +%Y-%m-%d)
HANDOFF_FILE=".orbit/HANDOFF-${PROJECT}-${TIMESTAMP}.md"
```

**Write content (NOT from template, populate directly):**

```markdown
# ORBIT Handoff

**Date:** [current timestamp]
**Status:** [paused/blocked/context-limit]

---

## READ THIS FIRST

You have no prior context. This document tells you everything.

**Project:** [from PROJECT.md]
**Core value:** [from PROJECT.md]

---

## Current State

**Version:** [from STATE.md]
**Project:** [N] of [total] — [project name]
**Plan:** [plan-id] — [status]

**Loop Position:**
```
REFINE ──▶ BUILD ──▶ INTEGRATE
  [✓/○]    [✓/○]    [✓/○]
```

---

## What Was Done

- [Accomplishment 1]
- [Accomplishment 2]
- [Accomplishment 3]

---

## What's In Progress

- [In-progress item with status]

---

## What's Next

**Immediate:** [specific next action]

**After that:** [following action]

---

## Key Files

| File | Purpose |
|------|---------|
| `.orbit/STATE.md` | Live project state |
| `.orbit/ROADMAP.md` | Project overview |
| [current plan path] | [plan purpose] |

---

## Resume Instructions

1. Read `.orbit/STATE.md` for latest position
2. Check loop position
3. Run `/orbit:resume` or `/orbit:progress`

---

*Handoff created: [timestamp]*
```

Be specific enough for a fresh Claude to understand immediately.
</step>

<step name="update_state">
**Update STATE.md — two sections:**

**1. Projects Overview — mark current project as Paused:**
- Find the current project row
- Change Status from `🔵 In Progress` → `⏸ Paused`
- Loop position column stays as-is (preserves exact position)
- This allows another project to take `🔵 In Progress`

**2. Session Continuity:**
```markdown
## Session Continuity

Last session: [timestamp]
Stopped at: [what was happening — project + loop position]
Next action: /orbit:resume — choose which project to continue
Resume file: .orbit/HANDOFF-[project]-[date].md
```
</step>

<step name="optional_commit">
**If git repo, offer WIP commit with explicit two-question flow:**

**Question 1 — Commit WIP?**
```
────────────────────────────────────────
Would you like to commit your work-in-progress?

This saves a checkpoint you can return to.
────────────────────────────────────────
[yes] / [no]
```

**If no:** Skip to confirm step.

**If yes — Question 2 — Branch choice:**
```
────────────────────────────────────────
Where should this WIP commit go?

[1] main — Commit directly to main branch
[2] feature branch — Create feature/{project-name} branch first

Note: Feature branch is useful if work isn't ready for main.
────────────────────────────────────────
```

**If main (option 1):**
```bash
git add .orbit/ src/
git commit -m "wip({project}): paused at {plan}

Co-Authored-By: Claude <noreply@anthropic.com>"
```

**If feature branch (option 2):**
```bash
# Create and switch to feature branch
git checkout -b feature/{project-name}

# Commit to feature branch
git add .orbit/ src/
git commit -m "wip({project}): paused at {plan}

Co-Authored-By: Claude <noreply@anthropic.com>"

# Record branch strategy in STATE.md for transition-phase
```

**Update STATE.md Session Continuity with branch info:**
```markdown
Git strategy: {main|feature/{project-name}}
```

This enables transition-phase.md to know the branch strategy when reconciling.
</step>

<step name="confirm">
**Display confirmation:**

```
════════════════════════════════════════
ORBIT SESSION PAUSED
════════════════════════════════════════

Handoff: .orbit/HANDOFF-[project]-[date].md

Project [N] — [Name]
  Status: ⏸ Paused
  Loop: REFINE [✓/○] → BUILD [✓/○] → INTEGRATE [✓/○]

To continue this project later:
  /orbit:resume → select [project name]

To start another project now:
  /orbit:refine (for next project in ROADMAP)

════════════════════════════════════════
```
</step>

</process>

<output>
- HANDOFF-{date}.md created in .orbit/
- STATE.md updated with session continuity
- Optional WIP commit with branch choice (main or feature/{project})
- Git strategy recorded in STATE.md for transition-phase
- User knows how to resume
</output>

<error_handling>
**No .orbit/ directory:**
- "No ORBIT project found. Nothing to pause."

**STATE.md missing or corrupted:**
- Create minimal handoff from available context
- Note the gap in handoff file

**Git not available:**
- Skip commit step, still create handoff
</error_handling>
