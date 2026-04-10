---
name: orbti:add-projeto
description: Create a new project folder — projetos contain milestones
argument-hint: "[projeto-name]"
allowed-tools: [Read, Write, Edit, Bash]
---

<model>haiku</model>

<objective>
Create a new project (projeto) folder inside `.orbti/projetos/`.
Milestones are created inside projects with `/orbti:milestone`.

**Hierarchy:** projeto → milestones → phases (refines)

**When to use:** Starting a new product, app, or domain area in the monorepo.
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/add-projeto.md
</execution_context>

<context>
$ARGUMENTS

@.orbti/STATE.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/add-projeto.md
</process>

<success_criteria>
- [ ] `.orbti/projetos/{slug}/` created
- [ ] `.orbti/projetos/{slug}/ROADMAP.md` initialized
- [ ] STATE.md updated with new projeto
- [ ] User knows to run /orbti:milestone next
</success_criteria>
