<purpose>
Facilitate vision articulation before planning a project. Acts as a thinking partner to explore what the user wants to accomplish, then creates CONTEXT.md for handoff to refine.

**Philosophy:** Goals first — everything else (approach, constraints, risks) derives from what the user wants to achieve.

**Distinction from assumptions:** This workflow gathers USER input about the project. The assumptions workflow surfaces CLAUDE's understanding for validation.
</purpose>

<when_to_use>
- User starting a new project and wants to think through approach
- User has rough ideas but needs to articulate them before planning
- Before /orbti:refine when discussion is desired
- Project scope is unclear or has multiple possible approaches
</when_to_use>

<loop_context>
N/A - This is a pre-planning workflow, not a loop phase.
After discussion, routes to /orbti:refine (refine).
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/PROJECT.md (current requirements and progress)
@.orbti/ROADMAP.md (phase scope and goals)
</required_reading>

<references>
@src/templates/CONTEXT.md (handoff format)
</references>

<process>

<step name="validate_phase" priority="first">
1. Parse phase number from $ARGUMENTS

**If argument missing:**
```
Error: Project number required.

Usage: /orbti:cocreate <project-number>
Example: /orbti:cocreate 10
```
Exit workflow.

2. Validate project exists in ROADMAP.md
3. Extract project details: number, name, description, scope

**If project not found:**
```
Error: Project {N} not found in roadmap.

Available projects:
[list incomplete projects from roadmap]
```
Exit workflow.
</step>

<step name="phase_context">
**Present project context:**

```
════════════════════════════════════════
PROJECT DISCUSSION
════════════════════════════════════════

Project: {project_number} — {project_name}
Status: {from ROADMAP.md}

Roadmap description:
{project description from ROADMAP.md}

{If prior project completed:}
Prior project: {prior_project_name}
What was built: {summary}

────────────────────────────────────────
```

This gives user context for the discussion.
</step>

<step name="explore_goals">
**The core question — goals first:**

```
What do you want to accomplish in this project?

Don't worry about implementation details yet — just describe what
success looks like and any specific goals you have in mind.

(Examples: "Get research workflows working", "Make the CLI intuitive",
 "Ensure proper error handling")
```

Wait for user response.

**Follow-up if needed:**
- "Any specific features or capabilities you're prioritizing for this project?"
- "What's the most important outcome?"
- "Any concerns or risks you want to address?"

Store responses as `goals` list.
</step>

<step name="explore_approach">
**Ask about approach:**

```
How do you want to approach this phase?

Consider:
- Any specific patterns or libraries to use/avoid?
- Constraints or limitations?
- Dependencies on other work?
```

Wait for user response.

Store responses as `approach` notes.
</step>

<step name="synthesize_context">
From the discussion, derive:

1. **Key goals:**
   - Synthesize main objectives from discussion
   - Confirm: "So the main goals are: {goals}. Sound right?"

2. **Approach notes:**
   - Capture technical direction
   - Note any constraints mentioned

3. **Open questions:**
   - Identify anything still unclear
   - Note items to research or decide during planning

Confirm with user before proceeding.
</step>

<step name="write_context">
Create `.orbti/projects/{NN}-{name}/CONTEXT.md`:

Use CONTEXT.md template format.

Display:
```
Context saved to .orbti/projects/{NN}-{name}/CONTEXT.md

This file persists across /clear so you can take a break if needed.
```
</step>

<step name="handoff">
Route to refine:

```
════════════════════════════════════════
DISCUSSION COMPLETE
════════════════════════════════════════

Project: {project_number} — {project_name}
Goals: {goal_count}
Status: Ready for planning

Context saved for handoff.

────────────────────────────────────────
▶ NEXT: /orbti:refine
  Create the refine structure from this context
────────────────────────────────────────

Type "yes" to proceed, or continue discussing.
```

**Accept:** "yes", "go", "refine" → run `/orbti:refine`
</step>

</process>

<output>
- .orbti/projects/{NN}-{name}/CONTEXT.md created (handoff file)
- Goals and approach articulated
- Ready for /orbti:refine to consume
</output>

<success_criteria>
- [ ] Project validated against ROADMAP.md
- [ ] Project context presented
- [ ] Goals explored (user-driven)
- [ ] Approach discussed
- [ ] Context synthesized and confirmed
- [ ] CONTEXT.md written to project directory
- [ ] Clear handoff to /orbti:refine
</success_criteria>

<anti_patterns>
**Asking abstract questions first:**
DON'T: "What's the scope of this project?"
DO: "What do you want to accomplish?"

**Assuming approach before goals:**
DON'T: "What libraries will you use?"
DO: Derive approach from goals discussed.

**Skipping confirmation:**
DON'T: Write context immediately after one question.
DO: Confirm the synthesis makes sense before writing.

**Not persisting context:**
DON'T: End discussion without writing CONTEXT.md
DO: Always write the file so /clear doesn't lose progress.

**Duplicating assumptions workflow:**
DON'T: Surface Claude's assumptions here.
DO: Focus on user input. Use /orbti:assumptions for Claude's analysis.
</anti_patterns>

<error_handling>
**User unsure what to accomplish:**
- Reference ROADMAP.md project description
- Suggest: "Based on the roadmap, this project should..."
- Offer: "We can start with the roadmap goals and refine"

**Scope too large:**
- Suggest splitting into multiple refines
- Ask: "What's the most critical part?"

**User wants to skip discussion:**
- Route directly to /orbti:refine
- Note: "Going straight to planning — no discussion context will be available"
</error_handling>
