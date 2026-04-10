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
@.orbti/STATE.md
@.orbti/PROJECT.md
</required_reading>

<references>
@./.claude/orbti-framework/references/context-management.md
@./.claude/orbti-framework/templates/HANDOFF.md
</references>

<process>

<step name="detect_position" priority="first">
1. Read `.orbti/STATE.md` Projects Overview
2. Count rows with status `🔵 In Progress` or `⏸ Paused`

**If multiple active projects and no argument provided:**
```
════════════════════════════════════════
Which project do you want to pause?
════════════════════════════════════════

│  01  auth-service    🔵 In Progress  ✓ ◉ ○  │
│  02  dashboard       🔵 In Progress  ◉ ○ ○  │

Type the project number or name (e.g., "01" or "auth").
════════════════════════════════════════
```
Wait for selection, then proceed with that project.

**If argument provided** (e.g. `/orbti:pause 01` or `/orbti:pause auth`):
- Resolve to matching project row in Overview
- If not found: "Project '[arg]' not found. Active projects: [list]"

**If only one active project:** proceed directly with that project.
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
**Create HANDOFF file — inside the project folder:**

```bash
# Handoff lives in the current project's directory
PROJECT_DIR=$(ls -d .orbti/projects/*/ 2>/dev/null | sort | tail -1)  # current project folder
TIMESTAMP=$(date +%Y-%m-%d)
HANDOFF_FILE="${PROJECT_DIR}HANDOFF-${TIMESTAMP}.md"
# e.g. .orbti/projects/01-auth/HANDOFF-2026-03-17.md
```

**Write content (NOT from template, populate directly):**

```markdown
# ORBTI Handoff

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
**Refine:** [refine-id] — [status]

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
| `.orbti/STATE.md` | Live project state |
| `.orbti/ROADMAP.md` | Project overview |
| [current refine path] | [refine purpose] |

---

## Resume Instructions

1. Read `.orbti/STATE.md` for latest position
2. Check loop position
3. Run `/orbti:resume` or `/orbti:progress`

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
Next action: /orbti:resume — choose which project to continue
Resume file: .orbti/projects/[project]/HANDOFF-[date].md
```
</step>

<step name="capture_to_napkin">
**Capture session learnings to napkin before pausing:**

1. Check if `.orbti/napkin.md` exists — if not, create it with the structure below
2. Append a session entry with what happened this session:

```markdown
## [YYYY-MM-DD HH:MM] Session — {project}/{refine}

### What Was Done
- [Accomplishment 1]
- [Accomplishment 2]

### Corrections & Redirects
- [Any time user corrected approach or redirected]
  Do instead: [concrete repeatable action]

### Surprises
- [Unexpected tool/command behavior, env quirks]
  Do instead: [concrete repeatable action]

### What Worked Well
- [Technique or approach worth repeating]
```

3. After appending, curate the file:
   - Keep only recurring, high-value entries (skip one-off events with no pattern)
   - Each entry with `Do instead:` must be actionable and specific
   - Remove entries older than 30 days that haven't recurred
   - Max 15 session entries total (remove oldest if exceeded)

4. This file is consumed by `/orbti:integrate` to reconstruct what happened — write clearly enough for a fresh agent to understand.

If nothing notable happened (clean session, no corrections, no surprises): write a minimal entry with just "What Was Done".
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
git add .orbti/ src/
git commit -m "wip({project}): paused at {refine}

Co-Authored-By: Claude <noreply@anthropic.com>"
```

**If feature branch (option 2):**
```bash
# Create and switch to feature branch
git checkout -b feature/{project-name}

# Commit to feature branch
git add .orbti/ src/
git commit -m "wip({project}): paused at {refine}

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
**Display confirmation block:**

```
════════════════════════════════════════
ORBTI SESSION PAUSED
════════════════════════════════════════

Handoff: .orbti/projects/[project]/HANDOFF-[date].md

[Name]
  Status: ⏸ Paused
  Loop: REFINE [✓/○] → BUILD [✓/○] → INTEGRATE [✓/○]

[If other pending projects, list them here]
════════════════════════════════════════
```

Then use AskUserQuestion:
```
question: "O que fazer agora?"
header: "Próximo passo"
options:
  - label: "Encerrar"
    description: "Terminar a sessão — retomar depois com /orbti:resume"
  - label: "Continuar projeto"
    description: "Ir para o próximo passo deste projeto agora"
  - label: "Outro projeto"
    description: "Trabalhar em outro projeto pendente"
```

**Route:**
- "Encerrar" → end session, show resume command
- "Continuar projeto" → run the next action from the handoff
- "Outro projeto" → list pending projects and ask which to resume
</step>

</process>

<output>
- HANDOFF-{date}.md created in .orbti/
- STATE.md updated with session continuity
- Optional WIP commit with branch choice (main or feature/{project})
- Git strategy recorded in STATE.md for transition-phase
- User knows how to resume
</output>

<error_handling>
**No .orbti/ directory:**
- "No ORBTI project found. Nothing to pause."

**STATE.md missing or corrupted:**
- Create minimal handoff from available context
- Note the gap in handoff file

**Git not available:**
- Skip commit step, still create handoff
</error_handling>
