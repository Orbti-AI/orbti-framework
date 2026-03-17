<purpose>
Define a new milestone in the project. Creates milestone structure in ROADMAP.md, initializes project directories, and updates project state. Uses MILESTONE-CONTEXT.md handoff if available from cocreate-milestone.
</purpose>

<when_to_use>
- User explicitly requests new milestone
- Triggered after /orbit:cocreate-milestone (reads context)
- Project completed previous milestone, needs next
- Starting fresh project milestone planning
</when_to_use>

<loop_context>
N/A - This is a milestone setup workflow, not a loop phase.
After create-milestone, project is ready for first project REFINE.
</loop_context>

<required_reading>
@.orbit/STATE.md
@.orbit/ROADMAP.md
@.orbit/projects/{name}/MILESTONE-CONTEXT.md (if exists)
</required_reading>

<references>
@src/templates/ROADMAP.md (milestone section format)
@src/templates/milestone-context.md (handoff structure)
</references>

<process>

<step name="load_context" priority="first">
1. Check for MILESTONE-CONTEXT.md:
   ```bash
   cat .orbit/projects/{name}/MILESTONE-CONTEXT.md 2>/dev/null
   ```

2. **If found:**
   - Display: "Loading context from cocreate-milestone..."
   - Parse: Features, Scope, Phase Mapping, Constraints
   - Store for use in subsequent steps
   - Skip to step 4 (update_roadmap)

3. **If not found:**
   - Display: "No discussion context found. Let's define the milestone."
   - Proceed to step 2 (get_milestone_info)
</step>

<step name="get_milestone_info">
**Only if no MILESTONE-CONTEXT.md exists.**

Ask ONE question at a time:

**Question 1: Milestone name/version**
```
What version/name for this milestone?

Example: "v0.3 Roadmap Management" or "v1.0 Production Release"
```
Wait for response. Store as `milestone_name`.

**Question 2: Theme**
```
What's the focus of this milestone? (1 sentence)
```
Wait for response. Store as `milestone_theme`.
</step>

<step name="identify_phases">
**Only if no MILESTONE-CONTEXT.md exists.**

Ask about projects:

```
What projects will this milestone include?

(Example: "Templates, Workflows, Commands" or "3 projects for auth, dashboard, deployment")
```

Wait for response. Parse into project list.

For each project, derive:
- Project number (next available from ROADMAP.md)
- Project name
- Brief description
</step>

<step name="update_roadmap">
Read current ROADMAP.md and update:

1. **Add milestone header section:**
   ```markdown
   ## Current Milestone
   **{milestone_name}** ({version})
   Status: 🚧 In Progress
   Phases: 0 of {phase_count} complete
   ```

2. **Add projects to table:**
   ```markdown
   | Project | Name | Plans | Status | Completed |
   |---------|------|-------|--------|-----------|
   | {N} | {name} | TBD | Not started | - |
   ```

3. **Add project details section:**
   ```markdown
   ### Project {N}: {name}

   Focus: {description}
   Plans: TBD (defined during /orbit:refine)
   ```

4. **Update footer timestamp**
</step>

<step name="create_phase_directories">
For each phase in the new milestone:

```bash
mkdir -p .orbit/projects/{NN}-{name-slug}
```

Where:
- `NN` = zero-padded phase number
- `name-slug` = lowercase, hyphenated phase name
</step>

<step name="update_state">
Update STATE.md:

1. **Current Position:**
   ```markdown
   ## Current Position

   Milestone: {milestone_name}
   Project: {first_project_number} of {total} ({first_project_name})
   Plan: Not started
   Status: Ready to plan
   Last activity: {timestamp} — Milestone created
   ```

2. **Progress:**
   ```markdown
   Progress:
   - {milestone_name}: [░░░░░░░░░░] 0%
   ```

3. **Loop Position:**
   ```markdown
   ## Loop Position

   Current loop state:
   ```
   REFINE ──▶ BUILD ──▶ INTEGRATE
     ○        ○        ○     [Ready for first REFINE]
   ```
   ```

4. **Session Continuity:**
   ```markdown
   ## Session Continuity

   Last session: {timestamp}
   Stopped at: Milestone created, ready to plan
   Next action: /orbit:refine for Project {first_project_number}
   Resume file: .orbit/ROADMAP.md
   ```
</step>

<step name="cleanup_context">
**If MILESTONE-CONTEXT.md existed:**

Delete the handoff file:
```bash
rm .orbit/projects/{name}/MILESTONE-CONTEXT.md
```

Display: "Cleaned up milestone context handoff."

**Note:** This file is temporary — its job is done once milestone is created.
</step>

<step name="offer_next">
Display completion with ONE next action:

```
════════════════════════════════════════
MILESTONE CREATED
════════════════════════════════════════

Milestone: {milestone_name}
Theme: {milestone_theme}
Phases: {phase_count}

Created:
  .orbit/projects/{phase-1-slug}/     ✓
  .orbit/projects/{phase-2-slug}/     ✓
  .orbit/projects/{phase-N-slug}/     ✓

ROADMAP.md updated ✓
STATE.md updated ✓

────────────────────────────────────────
▶ NEXT: /orbit:refine
  Begin planning Project {first_project_number}: {first_project_name}
────────────────────────────────────────

Type "yes" to proceed, or ask questions first.
```

**Do NOT suggest multiple next steps.** ONE action only.
</step>

</process>

<output>
- ROADMAP.md updated with new milestone section
- Project directories created in .orbit/projects/
- STATE.md updated with new position
- MILESTONE-CONTEXT.md deleted (if existed)
- Clear routing to /orbit:refine
</output>

<success_criteria>
- [ ] MILESTONE-CONTEXT.md loaded if exists
- [ ] User prompted only if no context exists
- [ ] ROADMAP.md has new milestone section
- [ ] Project directories created
- [ ] STATE.md reflects new milestone position
- [ ] MILESTONE-CONTEXT.md cleaned up
- [ ] Single next action offered
</success_criteria>

<error_handling>
**MILESTONE-CONTEXT.md malformed:**
- Report parsing error
- Fall back to manual questions
- Clean up malformed file

**Project directory exists:**
- Check if empty → proceed
- If has content → warn about overwrite, ask to confirm

**ROADMAP.md missing:**
- Create basic structure
- Or route to /orbit:init if project not initialized
</error_handling>
