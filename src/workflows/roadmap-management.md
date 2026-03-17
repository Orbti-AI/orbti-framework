<purpose>
Handle dynamic roadmap modifications: adding projects to the current milestone and removing future projects. These are the "escape hatches" when scope changes mid-milestone.

**Operations:**
- **add-project:** Append a new project to the current milestone
- **remove-project:** Remove a future (not-started) project
</purpose>

<when_to_use>
- User realizes more projects are needed mid-milestone
- Scope changed and a planned project is no longer needed
- Emergency project insertion (use decimal projects for interruptions)
- Roadmap cleanup after requirements change
</when_to_use>

<loop_context>
N/A - This is a roadmap modification workflow, not a loop phase.
Can be invoked at any time during a milestone.
</loop_context>

<required_reading>
@.orbit/ROADMAP.md
@.orbit/STATE.md
</required_reading>

<references>
@src/templates/ROADMAP.md (project section format)
</references>

---

## Operation: add-project

<process>

<step name="add_validate" priority="first">
1. Read ROADMAP.md for current milestone
2. Identify highest project number in milestone
3. Calculate next project number = highest + 1

**If no active milestone:**
- Error: "No active milestone. Run /orbit:milestone first."
- Exit workflow
</step>

<step name="add_gather_info">
Ask for project details:

```
Adding project {next_number} to {milestone_name}.

Project name?
(Example: "Testing", "Documentation", "Performance")
```

Wait for response. Store as `project_name`.

Optional follow-up:
```
Brief description? (Press Enter to skip)
```

Store as `project_description` or derive from name.
</step>

<step name="add_update_roadmap">
Update ROADMAP.md:

1. **Update milestone header:**
   ```markdown
   Projects: {X} of {new_total} complete
   ```

2. **Add to projects table:**
   ```markdown
   | {next_number} | {project_name} | TBD | Not started | - |
   ```

3. **Add project details section:**
   ```markdown
   ### Project {next_number}: {project_name}

   Focus: {project_description}
   Plans: TBD (defined during /orbit:refine)
   Status: Not started
   ```

4. **Update footer timestamp**
</step>

<step name="add_create_directory">
Create project directory:

```bash
mkdir -p .orbit/projects/{NN}-{project-slug}
```

Where:
- `NN` = zero-padded project number
- `project-slug` = lowercase, hyphenated project name
</step>

<step name="add_update_state">
Update STATE.md:

1. **Last activity:**
   ```markdown
   Last activity: {timestamp} — Added Project {number}: {name}
   ```

2. **Decisions (add to table):**
   ```markdown
   | Added Project {number}: {name} | Project {current} | Extends milestone scope |
   ```
</step>

<step name="add_confirm">
Display confirmation:

```
════════════════════════════════════════
PROJECT ADDED
════════════════════════════════════════

Project {number}: {name}
Directory: .orbit/projects/{slug}/

{milestone_name} now has {total} projects.

ROADMAP.md updated ✓
STATE.md updated ✓

Continue with current work or plan this project later.
════════════════════════════════════════
```
</step>

</process>

---

## Operation: remove-project

<process>

<step name="remove_validate" priority="first">
1. Read ROADMAP.md for current milestone
2. Find target project by number or name
3. Check project status

**Validation rules:**
- Project must be "Not started" — cannot remove in-progress or complete projects
- Project must be in current milestone
- Cannot remove if it's the only remaining project

**If validation fails:**
```
Cannot remove Project {number}: {name}
Reason: {status is not "Not started" / only remaining project / not in current milestone}

Only future (not started) projects can be removed.
```
Exit workflow.
</step>

<step name="remove_confirm_intent">
Ask for confirmation:

```
════════════════════════════════════════
REMOVE PROJECT?
════════════════════════════════════════

Project {number}: {name}
Status: Not started

This will:
- Remove from ROADMAP.md
- Delete .orbit/projects/{slug}/ (if empty)
- Renumber subsequent projects

[1] Yes, remove | [2] Cancel
════════════════════════════════════════
```

Wait for confirmation. If "2" or "cancel" → exit.
</step>

<step name="remove_update_roadmap">
Update ROADMAP.md:

1. **Remove project from table**

2. **Remove project details section**

3. **Renumber subsequent projects:**
   - Project 7 removed → Project 8 becomes 7, Project 9 becomes 8, etc.
   - Update both table and details sections

4. **Update milestone header:**
   ```markdown
   Projects: {X} of {new_total} complete
   ```

5. **Update footer timestamp**
</step>

<step name="remove_cleanup_directory">
Handle project directory:

```bash
rmdir .orbit/projects/{NN}-{slug} 2>/dev/null
```

**If directory not empty:**
- Warn: "Directory not empty — preserved at .orbit/projects/{slug}/"
- User can manually delete if desired

**If directory empty or doesn't exist:**
- Silently removed or was never created
</step>

<step name="remove_renumber_directories">
**If subsequent projects exist:**

Renumber project directories to match ROADMAP.md:

```bash
mv .orbit/projects/08-name .orbit/projects/07-name
mv .orbit/projects/09-name .orbit/projects/08-name
```

**Note:** This is why we only allow removing "Not started" projects — they have no artifacts yet.
</step>

<step name="remove_update_state">
Update STATE.md:

1. **Last activity:**
   ```markdown
   Last activity: {timestamp} — Removed Project {number}: {name}
   ```

2. **Decisions (add to table):**
   ```markdown
   | Removed Project {original_number}: {name} | Project {current} | Scope reduction |
   ```

3. **If removed project was "next":**
   - Update current position to reflect new next project
</step>

<step name="remove_confirm">
Display confirmation:

```
════════════════════════════════════════
PROJECT REMOVED
════════════════════════════════════════

Removed: Project {original_number}: {name}
{renumbered_projects count} projects renumbered.

{milestone_name} now has {total} projects.

ROADMAP.md updated ✓
STATE.md updated ✓
════════════════════════════════════════
```
</step>

</process>

---

<output>
**add-project:**
- ROADMAP.md updated with new project
- Project directory created
- STATE.md decision logged

**remove-project:**
- ROADMAP.md updated (project removed, subsequent renumbered)
- Project directory removed (if empty)
- Subsequent directories renumbered
- STATE.md decision logged
</output>

<success_criteria>
**add-project:**
- [ ] Next project number calculated correctly
- [ ] Project added to ROADMAP.md table and details
- [ ] Directory created
- [ ] STATE.md updated
- [ ] Milestone total updated

**remove-project:**
- [ ] Project status validated (not started only)
- [ ] User confirmed removal
- [ ] Project removed from ROADMAP.md
- [ ] Directory cleaned up
- [ ] Subsequent projects renumbered
- [ ] STATE.md updated
</success_criteria>

<validation_rules>
**add-project:**
- Active milestone must exist
- Project name required
- Cannot add duplicate project names (warning only)

**remove-project:**
- Project must exist
- Project must be "Not started"
- Cannot remove last project in milestone
- Cannot remove project with artifacts (LOOP.md, INTEGRATE.md)
</validation_rules>

<error_handling>
**No active milestone:**
- Route to /orbit:milestone or /orbit:init

**Project directory not empty:**
- Preserve directory, warn user
- User can manually clean up

**Renumbering conflicts:**
- If directory already exists at target number, warn and skip
- Report which directories couldn't be renamed

**ROADMAP.md malformed:**
- Report parsing error
- Suggest manual fix
</error_handling>
