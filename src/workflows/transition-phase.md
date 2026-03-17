<purpose>
Handle project-level transition after all plans in a project are complete. Evolves PROJECT.md, verifies project completion, cleans up, and routes to next project or milestone completion.

**Invoked by:** integrate.md when it detects "last plan in project"
**Scope:** Project N → Project N+1 (or milestone completion)
</purpose>

<when_to_use>
- All plans in current project have INTEGRATE.md files
- Project is ready to close
- Moving to next project or completing milestone
</when_to_use>

<required_reading>
@.orbit/STATE.md
@.orbit/PROJECT.md
@.orbit/ROADMAP.md
@.orbit/projects/{current-phase}/*-INTEGRATE.md
</required_reading>

<process>

<step name="verify_phase_completion" priority="first">
1. Count LOOP.md files in current project directory
2. Count INTEGRATE.md files in current project directory
3. **Verification:**
   - If counts match: Project complete
   - If counts don't match: Project incomplete

**If incomplete:**
```
════════════════════════════════════════
PROJECT INCOMPLETE
════════════════════════════════════════

Project {N} has incomplete plans:
- {project}-01-INTEGRATE.md ✓
- {project}-02-INTEGRATE.md ✗ Missing
- {project}-03-INTEGRATE.md ✗ Missing

Options:
[1] Continue current project (execute remaining plans)
[2] Mark complete anyway (skip remaining plans)
[3] Review what's left
════════════════════════════════════════
```

Wait for user decision before proceeding.

**If complete:** Continue to next step.
</step>

<step name="cleanup_handoffs">
1. Check for stale handoffs in project directory:
   ```bash
   ls .orbit/projects/{current-project}/HANDOFF*.md 2>/dev/null
   ```
2. If found, delete them — project is complete, handoffs are stale
3. Note: Active handoffs at `.orbit/` root are preserved
</step>

<step name="evolve_project">
**Read project summaries:**
```bash
cat .orbit/projects/{current-project}/*-INTEGRATE.md
```

**Assess and update PROJECT.md:**

1. **Requirements validated?**
   - Any requirements shipped in this project?
   - Move to Validated section: `- ✓ [Requirement] — Project X`

2. **Requirements invalidated?**
   - Any requirements discovered unnecessary or wrong?
   - Move to Out of Scope: `- [Requirement] — [reason]`

3. **Requirements emerged?**
   - New requirements discovered during building?
   - Add to Active: `- [ ] [New requirement]`

4. **Key Decisions to log?**
   - Extract decisions from INTEGRATE.md files
   - Add to Key Decisions table

5. **Core value still accurate?**
   - If product meaningfully changed, update description
   - Keep it current

**Update footer:**
```markdown
---
*Last updated: [date] after Project [X]*
```
</step>

<step name="review_accumulated_context">
Update STATE.md Accumulated Context section:

**Decisions:**
- Note 3-5 recent decisions from this phase
- Full log lives in PROJECT.md

**Blockers/Concerns:**
- Resolved blockers: Remove from list
- Unresolved: Keep with "Project X" prefix
- New concerns from summaries: Add

**Deferred Issues:**
- Update count if issues were logged
- Note if many accumulated
</step>

<step name="update_state_for_transition">
Update STATE.md Current Position:

```markdown
## Current Position

Project: [N+1] of [total] ([Next project name])
Plan: Not started
Status: Ready to plan
Last activity: [today] — Project [N] complete, transitioned to Project [N+1]

Progress: [updated bar based on completed plans]
```

Update Session Continuity:
```markdown
## Session Continuity

Last session: [today]
Stopped at: Project [N] complete, ready to plan Project [N+1]
Next action: /orbit:refine for Project [N+1]
Resume file: .orbit/ROADMAP.md
```
</step>

<step name="update_roadmap_completion">
Update ROADMAP.md:

1. Mark current project complete:
   - Status: ✅ Complete
   - Completed: [date]
   - Plan count: X/X

2. Update progress summary:
   - Projects: Y of Z complete
   - Calculate percentage
</step>

<step name="commit_phase">
**Git commit for completed phase:**

**1. Check for feature branches from this project:**
```bash
git branch --list "feature/{project}*"
```

**2. If feature branch exists:**
```
────────────────────────────────────────
Feature branch detected: feature/{project-name}

Checking for conflicts with main...
────────────────────────────────────────
```

Check for conflicts:
```bash
git fetch origin main 2>/dev/null || true
git diff main...feature/{project-name} --stat
```

**If no conflicts:**
```
No conflicts detected.

Merge feature/{project-name} to main? [yes/no]
```

If yes:
```bash
git checkout main
git merge feature/{project-name} --no-ff -m "Merge feature/{project-name} into main"
git branch -d feature/{project-name}
```

**If conflicts exist:**
```
⚠️ Conflicts detected between feature/{project-name} and main.

Cannot auto-merge. Options:
[1] Resolve conflicts manually, then re-run transition
[2] Keep on feature branch (do not merge)
[3] Force merge anyway (not recommended)
```

**3. Stage project files:**
```bash
git add .orbit/projects/{project}/ .orbit/STATE.md .orbit/PROJECT.md .orbit/ROADMAP.md
git add src/  # If source files were modified
```

**4. Create project commit:**
```bash
git commit -m "$(cat <<'EOF'
feat({project}): {project-description}

Project {N} complete:
- {plan-01 summary}
- {plan-02 summary}
- {plan-03 summary}

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**5. Record git state for complete-milestone:**
Update STATE.md Accumulated Context:
```markdown
### Git State
Last commit: {short-hash}
Branch: main
Feature branches merged: {list or "none"}
```

Display:
```
Git commit created: {short-hash}
  feat({project}): {project-description}
```
</step>

<step name="verify_state_consistency" priority="critical">
**CRITICAL: Verify state files are aligned before declaring transition complete.**

State consistency is foundational to ORBIT. If STATE.md, PROJECT.md, or ROADMAP.md are misaligned, all downstream work breaks — resume fails, progress tracking is wrong, context is lost.

**1. Re-read all three files completely:**
```bash
cat .orbit/STATE.md
cat .orbit/PROJECT.md
cat .orbit/ROADMAP.md
```

**2. Verify alignment across these fields:**

| Field | STATE.md | PROJECT.md | ROADMAP.md |
|-------|----------|------------|------------|
| Version | `Version:` field | Current State table | Version Overview |
| Project | `Project:` field | (implicit in Active) | Project Structure table |
| Status | `Status:` field | `Status:` in table | Project status column |
| Focus | `Current focus:` header | (matches Active) | Current Milestone |

**3. Check for stale references:**
- No "blocked on X" if X is complete
- No "IN PROGRESS" for completed projects
- Current focus matches current project, not previous
- Progress bars match actual plan counts

**4. If ANY misalignment found:**
```
════════════════════════════════════════
⚠️ STATE CONSISTENCY ERROR
════════════════════════════════════════

Misalignment detected:
| Field | STATE.md | PROJECT.md | ROADMAP.md |
|-------|----------|------------|------------|
| {field} | {value} | {value} | {value} |

Fix ALL misalignments before proceeding.
This is a blocking error — do not route to next phase.
════════════════════════════════════════
```

**Fix the issues, then re-verify.**

**5. If aligned:**
```
State consistency: ✓
  STATE.md    — Project {N+1}, v{version}, ready to plan
  PROJECT.md  — v{version}, {active_count} active requirements
  ROADMAP.md  — Project {N} ✅, Project {N+1} 🔵
```

**Only proceed to route_next after verification passes.**
</step>

<step name="route_next">
**Check if milestone complete:**

1. Read ROADMAP.md
2. Find all projects in current milestone
3. If current project is LAST in milestone → Route B (milestone complete)
4. If more projects remain → Route A (next project)

---

**Route A: More projects remain**

```
════════════════════════════════════════
PROJECT {N} COMPLETE
════════════════════════════════════════

✓ All {X} plans complete
✓ PROJECT.md evolved
✓ Ready for next project

---
Next: Project {N+1} — {Name}

[1] Yes, plan Project {N+1} | [2] Pause here
════════════════════════════════════════
```

**Accept:** "1", "yes", "continue" → run `/orbit:refine` for Project N+1

---

**Route B: Milestone complete**

```
════════════════════════════════════════
MILESTONE COMPLETE
════════════════════════════════════════

🎉 {version} is 100% complete — all {N} projects finished!

✓ All phases unified
✓ PROJECT.md evolved
✓ Ready for next milestone or release

---
What's next?

[1] Start next milestone | [2] Review accomplishments | [3] Pause here
════════════════════════════════════════
```

</step>

</process>

<output>
- PROJECT.md evolved with validated/invalidated requirements
- STATE.md updated for new project
- ROADMAP.md marked complete
- Stale handoffs cleaned
- Git commit created for project: feat({project}): {description}
- Feature branches merged if applicable
- User routed to next project or milestone
</output>

<success_criteria>
- [ ] Project LOOP/SUMMARY count verified
- [ ] Stale handoffs cleaned
- [ ] PROJECT.md evolved (requirements, decisions)
- [ ] STATE.md updated (position, context, session)
- [ ] ROADMAP.md marked complete
- [ ] Feature branches merged (if any)
- [ ] Git commit created for project
- [ ] **STATE CONSISTENCY VERIFIED** (all three files aligned - BLOCKING)
- [ ] User knows next steps with quick continuation
</success_criteria>
