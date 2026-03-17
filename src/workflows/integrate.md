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
Next phase: REFINE (next plan or next phase)
</loop_context>

<required_reading>
@.orbit/STATE.md
@.orbit/phases/{phase}/{plan}-REFINE.md
</required_reading>

<references>
@~/.claude/orbit-framework/references/loop-phases.md
@~/.claude/orbit-framework/templates/INTEGRATE.md
@~/.claude/orbit-framework/workflows/transition-phase.md (loaded when last plan in phase)
</references>

<process>

<step name="gather_results" priority="first">
1. Recall execution from BUILD phase:
   - Which tasks completed successfully
   - Which tasks failed (if any)
   - Which checkpoints were resolved and how
   - Any deviations from the plan
2. Read REFINE.md to refresh:
   - Original acceptance criteria
   - Expected outputs
   - Task definitions
</step>

<step name="compare_plan_vs_actual">
1. For each acceptance criterion:
   - Was it satisfied? (PASS/FAIL)
   - Evidence of satisfaction
2. For each task:
   - Did it complete as specified?
   - Any modifications to approach?
3. Note deviations:
   - What differed from plan
   - Why it differed
   - Impact on outcomes
</step>

<step name="audit_skill_invocations">
**Check specialized workflow usage (if configured):**

1. Check if .orbit/SPECIAL-FLOWS.md exists:
   ```bash
   ls .orbit/SPECIAL-FLOWS.md 2>/dev/null
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

<step name="create_summary">
1. Create INTEGRATE.md at `.orbit/phases/{phase}/{plan}-INTEGRATE.md`
2. Include:

   **Frontmatter:**
   ```yaml
   ---
   phase: NN-name
   plan: NN
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

<step name="update_state">
1. Update STATE.md:

   **Current Position:**
   - Phase: N of M - Complete (or In Progress if more plans)
   - Plan: complete
   - Status: Ready for next REFINE
   - Last activity: timestamp

   **Progress:**
   - Update milestone percentage
   - Update phase percentage (100% if complete)

   **Loop Position:**
   ```
   REFINE ──▶ BUILD ──▶ INTEGRATE
     ✓        ✓        ✓     [Loop complete - ready for next REFINE]
   ```

   **Session Continuity:**
   - Update stopped at
   - Update next action
   - Update resume file (point to SUMMARY)
</step>

<step name="check_phase_completion">
**Determine if this is the last plan in the phase:**

1. Count REFINE.md files in current phase directory
2. Count INTEGRATE.md files (including the one just created)
3. Compare counts:
   - If REFINE count = SUMMARY count → Last plan, trigger transition
   - If REFINE count > SUMMARY count → More plans remain in phase
</step>

<step name="route_based_on_completion">
**If more plans remain in phase:**

Report with quick continuation:
```
════════════════════════════════════════
LOOP COMPLETE
════════════════════════════════════════

Plan: {NN}-{plan} — [description]
[summary of what was built]
[deviations if any]

Phase {N} progress: {X}/{Y} plans complete

---
Continue to next plan?

[1] Yes, plan {NN+1} | [2] Pause here
════════════════════════════════════════
```

**Accept:** "1", "yes", "continue" → run `/orbit:refine` for next plan in same phase
</step>

<step name="execute_transition" priority="required" gate="blocking">
**If last plan in phase — TRANSITION IS MANDATORY:**

⚠️ **NEVER skip this step. This is what makes ORBIT a system, not random loops.**

1. Announce clearly:
   ```
   ════════════════════════════════════════
   PHASE {N} COMPLETE — TRANSITION REQUIRED
   ════════════════════════════════════════
   ```

2. **MUST read and execute:** @~/.claude/orbit-framework/workflows/transition-phase.md

3. Transition handles (do not skip any):
   - Evolve PROJECT.md (requirements validated → shipped)
   - Update ROADMAP.md (phase status → complete)
   - Git commit for phase: `feat({phase}): {description}`
   - Clean stale handoffs
   - Route to next phase or milestone completion

4. **Only after transition completes** → offer next phase routing

**Anti-pattern:** Closing INTEGRATE and immediately offering `/orbit:refine` for next phase WITHOUT running transition. This breaks system cohesion and skips git commits.
</step>

</process>

<output>
- INTEGRATE.md at `.orbit/phases/{phase}/{plan}-INTEGRATE.md`
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
- Cannot reconcile without original plan
- Ask user to locate or reconstruct
</error_handling>

<anti_patterns>
**Skipping SUMMARY:**
Every completed plan MUST have a INTEGRATE.md. No exceptions.

**Partial state updates:**
Update ALL of: SUMMARY, STATE, ROADMAP (if phase done). Don't leave partial.

**Vague summaries:**
"It worked" is not a summary. Document files, AC results, deviations specifically.

**Forgetting loop position:**
Always show the visual loop position in STATE.md.
</anti_patterns>
