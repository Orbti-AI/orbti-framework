<purpose>
Create an executable LOOP.md for the current or specified project. The plan defines objective, acceptance criteria, tasks, boundaries, and verification - everything needed for BUILD phase execution.
</purpose>

<when_to_use>
- Starting a new project (ROADMAP shows next project ready)
- Previous plan completed (loop closed with INTEGRATE)
- First plan in a project (after init-project)
- Resuming work that needs a new plan
</when_to_use>

<loop_context>
Expected phase: REFINE
Prior phase:  INTEGRATE (previous plan complete) or none (first plan)
Next project: BUILD (after plan approval)
</loop_context>

<required_reading>
@.orbit/STATE.md
@.orbit/ROADMAP.md
@.orbit/PROJECT.md
@.orbit/projects/{prior-project}/{plan}-INTEGRATE.md (if exists and relevant)
</required_reading>

<references>
@~/.claude/orbit-framework/references/plan-format.md
@~/.claude/orbit-framework/references/checkpoints.md (if plan will have checkpoints)
@~/.claude/orbit-framework/templates/REFINE.md
</references>

<process>

<step name="validate_preconditions" priority="first">
1. Read STATE.md to confirm:
   - Loop position is ready for REFINE (prior INTEGRATE complete or first plan)
   - No blockers preventing planning
2. If STATE.md shows mid-loop (BUILD or INTEGRATE incomplete):
   - Warn user: "Previous loop not closed. Complete INTEGRATE first or reset."
   - Do not proceed until resolved
</step>

<step name="identify_phase">
1. Read ROADMAP.md to determine:
   - Which project is next (first incomplete project)
   - Project scope and goals
   - Dependencies on prior projects
2. If multiple projects available, ask user which to plan
3. Confirm project selection before proceeding
</step>

<step name="analyze_scope">
1. Review project goals from ROADMAP.md
2. Estimate number of tasks needed:
   - Target: 2-3 tasks per plan
   - If >3 tasks, consider splitting into multiple plans
3. Identify files that will be modified
4. Determine if checkpoints are needed:
   - Visual verification required? → checkpoint:human-verify
   - Architecture decision needed? → checkpoint:decision
   - Unavoidable manual action? → checkpoint:human-action (rare)
5. Set autonomous flag: true if no checkpoints, false otherwise
</step>

<step name="load_context">
1. Read PROJECT.md for:
   - Core requirements and constraints
   - Value proposition (what matters)
2. If prior project exists, read its INTEGRATE.md for:
   - What was built
   - Decisions made
   - Any deferred issues
3. **Check for LEARNINGS.md:**
   ```bash
   ls .orbit/LEARNINGS.md 2>/dev/null
   ```
   If exists: read `.orbit/LEARNINGS.md` and extract patterns relevant to the work being planned.
   Apply learnings actively — if a past correction matches a task being planned, adjust the task to avoid the known failure mode.
   If no LEARNINGS.md: skip.
4. Read source files relevant to this phase's work
5. Do NOT reflexively chain all prior summaries - only load what's genuinely needed
</step>

<step name="check_specialized_flows">
**Check for SPECIAL-FLOWS.md and populate skills section.**

1. Check if `.orbit/SPECIAL-FLOWS.md` exists
2. If exists:
   - Read SPECIAL-FLOWS.md
   - Extract skills marked as "required" for the work type being planned
   - Match against phase/plan work being done
   - Prepare <skills> section content for LOOP.md
3. If not exists:
   - Add comment: "No SPECIAL-FLOWS.md - skills section omitted"
   - Skip skills section in REFINE (or include minimal placeholder)
4. Display reminder if required skills found:
   ```
   ════════════════════════════════════════
   ⚠️  REQUIRED SKILLS for this plan:
   ════════════════════════════════════════
   - /skill-1 (work type: X)
   - /skill-2 (work type: Y)

   These must be loaded before /orbit:build will proceed.
   ════════════════════════════════════════
   ```

**Note:** The skills section is populated from SPECIAL-FLOWS.md during plan creation.
Required skills will BLOCK build until confirmed loaded.
</step>

<step name="create_plan">
1. Create project directory: `.orbit/projects/{NN}-{project-name}/`
2. Generate LOOP.md following template structure:

   **Frontmatter:**
   - project: NN-name
   - plan: 01 (or next number if multiple plans in phase)
   - type: execute (or tdd/research)
   - wave: 1 (adjust if dependencies exist)
   - depends_on: [] (or prior plan IDs if genuine dependency)
   - files_modified: [list all files]
   - autonomous: true/false

   **Sections:**
   - <objective>: Goal, Purpose, Output
   - <context>: @-references to project files and source
   - <acceptance_criteria>: Given/When/Then for each criterion
   - <tasks>: Task definitions with files, action, verify, done
   - <boundaries>: DO NOT CHANGE, SCOPE LIMITS
   - <verification>: Overall completion checks
   - <success_criteria>: Measurable completion
   - <output>: INTEGRATE.md location

3. Ensure every task has:
   - Clear files list
   - Specific action (not vague)
   - Verification command/check
   - Done criteria linking to AC-N
</step>

<step name="validate_plan">
1. Check all sections present
2. Verify acceptance criteria are testable
3. Confirm tasks are specific enough (files + action + verify + done)
4. Ensure boundaries protect completed work
5. Validate checkpoint placement (if any):
   - After automated work completes
   - Before dependent decisions
   - Not too frequent (avoid checkpoint fatigue)
</step>

<step name="update_state" priority="required">
**This step is REQUIRED. Do not skip.**

1. **Update STATE.md** with exact content:

   ```markdown
   ## Current Position

   Milestone: v0.1 [Milestone Name]
   Project: [N] of [total] ([Project Name]) — Planning
   Plan: [NN-PP] created, awaiting approval
   Status: REFINE created, ready for BUILD
   Last activity: [timestamp] — Created [plan-path]

   Progress:
   - Milestone: [░░░░░░░░░░] X%
   - Project [N]: [░░░░░░░░░░] 0%

   ## Loop Position

   Current loop state:
   ```
   REFINE ──▶ BUILD ──▶ INTEGRATE
     ✓        ○        ○     [Plan created, awaiting approval]
   ```

   ## Session Continuity

   Last session: [timestamp]
   Stopped at: Plan [NN-PP] created
   Next action: Review and approve plan, then run /orbit:build [plan-path]

   Resume file: [plan-path]
   ```

2. **Update ROADMAP.md** milestone status:
   - If first plan of milestone: Change "Not started" → "In progress"
   - Update project status: "Not started" → "Planning"

3. **Report with quick continuation prompt:**
   ```
   ════════════════════════════════════════
   REFINE CREATED
   ════════════════════════════════════════

   Plan: [plan-path]
   Project: [N] — [Project Name]

   [plan summary - key tasks, checkpoints]

   ---
   Continue to BUILD?

   [1] Approved, run BUILD | [2] Questions first | [3] Pause here
   ```
4. **Accept quick inputs:** "1", "approved", "yes", "go" → run `/orbit:build [plan-path]`
</step>

</process>

<output>
LOOP.md at `.orbit/projects/{NN}-{project-name}/{NN}-{plan}-LOOP.md`

Example: `.orbit/projects/04-workflows-layer/04-01-LOOP.md`
</output>

<error_handling>
**STATE.md missing:**
- Offer to create from ROADMAP.md inference
- Or ask user to run init-project first

**ROADMAP.md missing:**
- Cannot plan without roadmap
- Ask user to create ROADMAP.md or run init-project

**Phase dependencies not met:**
- Warn user which prior phases must complete first
- Do not create plan until dependencies satisfied
</error_handling>
