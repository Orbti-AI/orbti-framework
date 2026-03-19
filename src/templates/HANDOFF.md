# ORBTI Handoff

**Date:** {{timestamp}}
**Session:** {{session_id}}
**Status:** {{status}}

---

## READ THIS FIRST

You have no prior context. This document tells you everything.

**Project:** {{project_name}}
**Core value:** {{core_value}}

---

## Current State

**Version:** {{version}}
**Project:** {{project_number}} of {{total_projects}} — {{project_name}}
**Refine:** {{refine_id}} — {{refine_status}}

**Loop Position:**
```
REFINE ──▶ BUILD ──▶ INTEGRATE
  {{refine_mark}}        {{apply_mark}}        {{unify_mark}}
```

---

## What Was Done

{{accomplished_list}}

---

## What's In Progress

{{in_progress_list}}

---

## What's Next

**Immediate:** {{next_action}}

**After that:** {{following_action}}

---

## Key Files

| File | Purpose |
|------|---------|
| `.orbti/STATE.md` | Live project state |
| `.orbti/ROADMAP.md` | Phase overview |
| {{current_refine_path}} | {{refine_purpose}} |

---

## Resume Instructions

1. Read `.orbti/STATE.md` for latest position
2. Check if REFINE refine exists for current phase
3. Based on loop position:
   - `○○○` (fresh) → Run `/orbti:refine`
   - `✓○○` (planned) → Review refine, then `/orbti:build`
   - `✓✓○` (applied) → Run `/orbti:integrate`
   - `✓✓✓` (complete) → Ready for next project

**Or simply run:** `/orbti:resume`

---

*Handoff created: {{timestamp}}*
*This file is the single entry point for fresh sessions*
