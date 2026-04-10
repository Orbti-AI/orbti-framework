<purpose>
Create a new milestone inside a projeto folder. Milestones are the development cycles inside a project.

**Hierarchy:**
```
.orbti/projetos/{projeto}/
  ROADMAP.md
  milestones/
    {milestone-slug}/      ← this workflow creates this
      01-{phase}/
      02-{phase}/
```

Uses MILESTONE-CONTEXT.md handoff if available from observe-milestone.
</purpose>

<when_to_use>
- User explicitly requests new milestone
- Triggered after /orbti:observe-milestone (reads context)
- Projeto completed previous milestone, needs next cycle
- First milestone in a new projeto
</when_to_use>

<loop_context>
N/A - This is a milestone setup workflow, not a loop phase.
After create-milestone, projeto is ready for first REFINE.
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/projetos/{projeto}/ROADMAP.md (if exists)
@.orbti/projetos/{projeto}/milestones/{name}/MILESTONE-CONTEXT.md (if exists)
</required_reading>

<references>
@src/templates/ROADMAP.md (milestone section format)
@src/templates/milestone-context.md (handoff structure)
</references>

<process>

<step name="load_context" priority="first">
1. Check for MILESTONE-CONTEXT.md:
   ```bash
   cat .orbti/projects/{name}/MILESTONE-CONTEXT.md 2>/dev/null
   ```

2. **If found:**
   - Display: "Loading context from observe-milestone..."
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
Update `.orbti/projetos/{projeto_slug}/ROADMAP.md`:

1. **Update Milestones table:**
   ```markdown
   | {milestone_name} | 🚧 In Progress | 0/{phase_count} | - |
   ```

2. **Add Active Milestone section:**
   ```markdown
   ## Active Milestone: {milestone_name} ({version})

   **Goal:** {milestone_theme}
   **Status:** Phase 1 of {phase_count}

   | Phase | Name | Refines | Status | Completed |
   |-------|------|---------|--------|-----------|
   | {N} | {name} | TBD | Not started | - |
   ```

3. **Add phase details:**
   ```markdown
   ### Phase {N}: {name}

   Focus: {description}
   Refines: TBD (defined during /orbti:refine)
   ```

4. **Update footer timestamp**
</step>

<step name="select_projeto">
**Determine which projeto this milestone belongs to:**

1. Check STATE.md for a single active projeto — if only one, use it directly
2. If multiple projetos exist, ask:
   ```
   Which projeto for this milestone?

   [list projetos from .orbti/projetos/]
   ```
3. If no projetos exist:
   - Error: "No projetos found. Run /orbti:add-projeto first."
   - Exit workflow

Store as `projeto_slug`.
</step>

<step name="create_phase_directories">
Create milestone folder and initial phase directories:

```bash
# Milestone folder inside the projeto
mkdir -p .orbti/projetos/{projeto_slug}/milestones/{milestone_slug}

# Phase directories inside the milestone
mkdir -p .orbti/projetos/{projeto_slug}/milestones/{milestone_slug}/{NN}-{name-slug}
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
   Refine: Not started
   Status: Ready to refine
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
   Stopped at: Milestone created, ready to refine
   Next action: /orbti:refine for Project {first_project_number}
   Resume file: .orbti/ROADMAP.md
   ```
</step>

<step name="cleanup_context">
**If MILESTONE-CONTEXT.md existed:**

Delete the handoff file:
```bash
rm .orbti/projects/{name}/MILESTONE-CONTEXT.md
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
  .orbti/projects/{phase-1-slug}/     ✓
  .orbti/projects/{phase-2-slug}/     ✓
  .orbti/projects/{phase-N-slug}/     ✓

ROADMAP.md updated ✓
STATE.md updated ✓

────────────────────────────────────────
▶ NEXT: /orbti:refine
  Begin planning Project {first_project_number}: {first_project_name}
────────────────────────────────────────

Type "yes" to proceed, or ask questions first.
```

**Do NOT suggest multiple next steps.** ONE action only.
</step>

</process>

<output>
- ROADMAP.md updated with new milestone section
- Project directories created in .orbti/projects/
- STATE.md updated with new position
- MILESTONE-CONTEXT.md deleted (if existed)
- Clear routing to /orbti:refine
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
- Or route to /orbti:init if project not initialized
</error_handling>
