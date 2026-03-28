<purpose>
Reconcile what was planned vs. what was built. Create INTEGRATE.md documenting results, update STATE.md with new position, and close the loop to prepare for next REFINE.
</purpose>

<when_to_use>
- BUILD phase complete (all tasks executed or documented)
- Ready to close the current loop
- Need to record what was built for future reference
</when_to_use>

<loop_context>
Expected phase: INTEGRATE
Prior phase: BUILD (execution complete)
Next phase: REFINE (next refine or next phase)
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/projects/{project}/{refine}-REFINE.md
</required_reading>

<references>
@~/.claude/orbti-framework/references/loop-phases.md
@~/.claude/orbti-framework/templates/INTEGRATE.md
@~/.claude/orbti-framework/workflows/transition-phase.md (loaded when last refine in phase)
</references>

<process>

<step name="gather_results" priority="first">
1. Recall execution from BUILD phase:
   - Which tasks completed successfully
   - Which tasks failed (if any)
   - Which checkpoints were resolved and how
   - Any deviations from the refine
2. Read REFINE.md to refresh:
   - Original acceptance criteria
   - Expected outputs
   - Task definitions
</step>

<step name="compare_refine_vs_actual">
1. For each acceptance criterion:
   - Was it satisfied? (PASS/FAIL)
   - Evidence of satisfaction
2. For each task:
   - Did it complete as specified?
   - Any modifications to approach?
3. Note deviations:
   - What differed from refine
   - Why it differed
   - Impact on outcomes
</step>

<step name="audit_skill_invocations">
**Check specialized workflow usage (if configured):**

1. Check if .orbti/SPECIAL-FLOWS.md exists:
   ```bash
   ls .orbti/SPECIAL-FLOWS.md 2>/dev/null
   ```

2. If not exists: Skip this step entirely

3. If exists:
   a. Read SPECIAL-FLOWS.md
   b. For each skill with priority "required":
      - Check if skill was invoked during this BUILD phase
      - Mark as ✓ (invoked) or ○ (gap)
   c. If any gaps found:
      - Add to STATE.md Deviations section:
        ```markdown
        ### Skill Audit (Phase [N])
        | Expected | Invoked | Notes |
        |----------|---------|-------|
        | /skill-name | ○ | [reason if known] |
        ```
      - Warn user: "Skill gap(s) documented in STATE.md. Review before next phase."
   d. Do NOT block INTEGRATE for skill gaps (warn only)

4. If all required skills invoked:
   - Note in INTEGRATE.md: "Skill audit: All required skills invoked ✓"

**Reference:** @references/specialized-workflow-integration.md
</step>

<step name="review_team_optional">
**Optional parallel review before closing the loop:**

Check if agent teams are enabled:
```bash
grep "agent_teams:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Teams are **off by default** — only active if `agent_teams.enabled: true` in `.orbti/config.md`.

**If teams enabled:**

Spawn a 3-reviewer team in parallel. Each reviewer has a distinct lens so they don't overlap:

```
Create a review team for refine: [refine-path]
Spawn 3 reviewers:
- code-reviewer: code quality, adherence to boundaries in REFINE.md, no unintended changes
- security-reviewer: input validation, exposed secrets, injection risks, auth issues
- coverage-reviewer: are all ACs actually satisfied? any edge cases missed?

Each reviewer reads the REFINE.md and the changed files.
Each produces a short findings list (PASS or issues with severity: blocker/major/minor).
Team lead synthesizes into a review summary.
```

After team completes, extract:
- Any **blocker** findings → block INTEGRATE, route to `/orbti:refine-fix`
- **Major** findings → include in INTEGRATE.md, user decides
- **Minor** findings → logged as deferred issues in INTEGRATE.md

**If teams not available:** skip this step, proceed to create_summary.

**If no blockers:** continue to create INTEGRATE.md including review summary.
</step>

<step name="create_summary">
1. Create INTEGRATE.md at `.orbti/projects/{phase}/{refine}-INTEGRATE.md`
2. Include:

   **Frontmatter:**
   ```yaml
   ---
   project: name
   refine: NN
   completed: ISO timestamp
   duration: approximate time
   ---
   ```

   **Sections:**
   - Objective (brief restatement)
   - What Was Built (table: file, purpose, lines)
   - Acceptance Criteria Results (table: AC, description, status)
   - Verification Results (command outputs)
   - Deviations (if any, with explanations)
   - Key Patterns/Decisions (if any emerged)
   - Next Phase (what comes next)
</step>

<step name="update_runbook">
**Update agent learning from this BUILD:**

1. Check if `.orbti/RUNBOOK.md` exists
2. If exists: curate — re-prioritize by impact, merge duplicates, remove stale entries
3. If not exists AND this BUILD had corrections, deviations, or notable patterns: create it
4. Add any entries logged during `execute_tasks` that weren't yet written
5. Rules:
   - Keep each category ≤ 10 items, highest impact first
   - Every entry must have a `Do instead:` line
   - Remove entries that are now obvious or no longer apply
   - Do NOT log one-off events or routine completions

Skip if BUILD was clean with no notable corrections or surprises.
</step>

<step name="update_state">
Update STATE.md — three sections:

**1. Projects Overview table:**
- Find the current project row
- Update loop position for this refine: `✓ ✓ ◉` → `✓ ✓ ✓`
- Increment loop count: `N/M` → `N+1/M`
- If more loops remain: keep Status as `🔵 In Progress`
- If last loop: Status → `✅ Complete` (transition-phase handles this)

**2. Current Focus:**
```
**Project:** [N] — [Name]
**Refine:** [A] complete
**Status:** Ready for next REFINE (or transitioning)
**Last activity:** [timestamp] — Refine [A] integrated

Loop position:
REFINE ──▶ BUILD ──▶ INTEGRATE
  ✓        ✓        ✓     [Loop complete - ready for next REFINE]
```

**3. Session Continuity:**
- Stopped at: Refine [A] integrated
- Next action: `/orbti:refine` for Refine [A+1] (or next project)
- Resume file: point to the INTEGRATE.md just created
</step>

<step name="check_phase_completion">
**Determine if this is the last refine in the phase:**

1. Count REFINE.md files in current project directory
2. Count INTEGRATE.md files (including the one just created)
3. Compare counts:
   - If REFINE count = SUMMARY count → Last refine, trigger transition
   - If REFINE count > SUMMARY count → More loops remain in project
</step>

<step name="route_based_on_completion">
**If more refines remain in phase:**

Report with quick continuation:
```
════════════════════════════════════════
REFINE COMPLETE
════════════════════════════════════════

Refine: {refine} — [description]
[summary of what was built]
[deviations if any]

Project {N} progress: {X}/{Y} loops complete

---
Continue to next loop?

[1] Yes, refine {NN+1} | [2] Pause here
════════════════════════════════════════
```

**Accept:** "1", "yes", "continue" → run `/orbti:refine` for next loop in same project
</step>

<step name="execute_transition" priority="required" gate="blocking">
**If last refine in phase — TRANSITION IS MANDATORY:**

⚠️ **NEVER skip this step. This is what makes ORBTI a system, not random loops.**

1. Announce clearly:
   ```
   ════════════════════════════════════════
   PROJECT {N} COMPLETE — TRANSITION REQUIRED
   ════════════════════════════════════════
   ```

2. **MUST read and execute:** @~/.claude/orbti-framework/workflows/transition-phase.md

3. Transition handles (do not skip any):
   - Evolve PROJECT.md (requirements validated → shipped)
   - Update ROADMAP.md (phase status → complete)
   - Git commit for phase: `feat({project-name}): {description}`
   - Clean stale handoffs
   - Route to next phase or milestone completion

4. **Only after transition completes** → offer next phase routing

**Anti-pattern:** Closing INTEGRATE and immediately offering `/orbti:refine` for next phase WITHOUT running transition. This breaks system cohesion and skips git commits.
</step>

</process>

<output>
- INTEGRATE.md at `.orbti/projects/{project-name}/{refine}-INTEGRATE.md`
- Updated STATE.md
- Updated ROADMAP.md (if phase complete)
</output>

<error_handling>
**BUILD not complete:**
- Check STATE.md for actual position
- If BUILD incomplete, cannot INTEGRATE
- Return to BUILD to complete or document blockers

**Missing execution context:**
- If no memory of execution results, read any logs
- Ask user to confirm what was completed
- Document uncertainty in INTEGRATE.md

**REFINE.md missing:**
- Cannot reconcile without original refine
- Ask user to locate or reconstruct
</error_handling>

<anti_patterns>
**Skipping SUMMARY:**
Every completed refine MUST have a INTEGRATE.md. No exceptions.

**Partial state updates:**
Update ALL of: SUMMARY, STATE, ROADMAP (if phase done). Don't leave partial.

**Vague summaries:**
"It worked" is not a summary. Document files, AC results, deviations specifically.

**Forgetting loop position:**
Always show the visual loop position in STATE.md.
</anti_patterns>
