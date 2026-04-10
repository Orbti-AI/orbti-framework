<purpose>
Create a new project (projeto) folder. Projects are the top-level containers in the ORBTI hierarchy.
Each projeto can have multiple milestones over time.

**Hierarchy:**
```
.orbti/projetos/{projeto}/
  ROADMAP.md
  milestones/
    {milestone-slug}/       ← created by /orbti:milestone
      01-{phase}/           ← created by /orbti:refine
      02-{phase}/
```
</purpose>

<when_to_use>
- Adding a new product, app, or domain to the monorepo
- Starting a new isolated workstream with its own roadmap
- User runs /orbti:add-projeto
</when_to_use>

<process>

<step name="get_projeto_info" priority="first">
**If argument provided** (e.g. `/orbti:add-projeto menos-juros`):
- Use argument as projeto name directly
- Skip question

**If no argument:**
```
What is the name of this project?

(Example: "menos-juros", "dashboard", "api-gateway")
```
Wait for response. Store as `projeto_name`.

Then ask:
```
Brief description? (1 sentence — what this project delivers)
```
Store as `projeto_description`.
</step>

<step name="derive_slug">
Derive slug from name:
- Lowercase
- Replace spaces and underscores with hyphens
- Example: "Menos Juros" → `menos-juros`

Store as `projeto_slug`.
</step>

<step name="check_existing">
Check if projeto already exists:
```bash
ls .orbti/projetos/{projeto_slug} 2>/dev/null
```

If exists:
```
Projeto '{projeto_slug}' already exists at .orbti/projetos/{projeto_slug}/

Use /orbti:milestone to add a new milestone to this project.
```
Exit workflow.
</step>

<step name="create_structure">
Create the project folder and initialize ROADMAP:

```bash
mkdir -p .orbti/projetos/{projeto_slug}/milestones
```

Create `.orbti/projetos/{projeto_slug}/ROADMAP.md`:

```markdown
# {projeto_name}

{projeto_description}

## Milestones

| Milestone | Status | Phases | Completed |
|-----------|--------|--------|-----------|

## Active Milestone

*(None yet — run /orbti:milestone to create the first milestone)*

---
*Created: {YYYY-MM-DD}*
*Last updated: {YYYY-MM-DD}*
```
</step>

<step name="update_state">
Update STATE.md — add projeto to Projects list:

1. If `## Projetos` section exists: add a row
2. If not exists: add the section

```markdown
## Projetos

| Projeto | Active Milestone | Status |
|---------|-----------------|--------|
| {projeto_name} | - | ○ Pending |
```
</step>

<step name="confirm">
Display confirmation:

```
════════════════════════════════════════
PROJETO CREATED
════════════════════════════════════════

Project: {projeto_name}
Folder:  .orbti/projetos/{projeto_slug}/

────────────────────────────────────────
▶ NEXT: /orbti:milestone
  Create the first milestone inside {projeto_name}
────────────────────────────────────────
```
</step>

</process>

<output>
- `.orbti/projetos/{slug}/` directory created
- `.orbti/projetos/{slug}/milestones/` directory created
- `.orbti/projetos/{slug}/ROADMAP.md` initialized
- STATE.md updated with new projeto
- User routed to /orbti:milestone
</output>

<error_handling>
**No .orbti/ directory:**
- "No ORBTI project initialized. Run /orbti:init first."

**projeto already exists:**
- Show existing path and suggest /orbti:milestone
</error_handling>
