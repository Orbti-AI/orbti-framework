---
name: orbti:milestone
description: Create a new milestone inside a projeto
argument-hint: "[milestone-name]"
allowed-tools: [Read, Write, Edit, Bash, Glob, AskUserQuestion]
---

<model>haiku</model>

<objective>
Create a new milestone inside a projeto folder.

**Hierarchy:** projeto → milestones → phases
**Folder:** `.orbti/projetos/{projeto}/milestones/{milestone}/`

**When to use:** Starting a new development cycle inside an existing projeto.
Requires a projeto created with /orbti:add-projeto.
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/create-milestone.md
</execution_context>

<context>
$ARGUMENTS

@.orbti/STATE.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/create-milestone.md
</process>

<success_criteria>
- [ ] Milestone folder created inside projeto
- [ ] projeto ROADMAP.md updated with milestone entry
- [ ] STATE.md reflects active milestone
- [ ] Phase directories initialized inside milestone
</success_criteria>
