---
name: orbti:refine
description: Enter REFINE phase. Detecta automaticamente frontend e backend — sempre em refines separados, sempre em paralelo. Aceita qualquer input de design (Figma, Pencil, Paper, HTML, descrição).
argument-hint: "[project or phase]"
allowed-tools: [Read, Write, Glob, AskUserQuestion, Skill]
---

<objective>
Create or continue a REFINE for the specified phase or project.

**Regra única:** frontend e backend são sempre refines separados, rodando em paralelo (loop 1, depends_on: []).

**Detecção automática:**
- Frontend: link Figma/Pencil/Paper/Stitch, arquivo HTML, código React, descrição de tela → `REFINE-FRONT.md`
- Backend: API, banco, serviço, lógica de negócio → `REFINE.md`
- Ambos → dois arquivos, mesma loop, paralelo

**Não há exceção:** se há trabalho dos dois lados, são sempre dois refines.
</objective>

<model>opus</model>

<execution_context>
@./.claude/orbti-framework/workflows/refine.md
@./.claude/orbti-framework/templates/REFINE.md
@./.claude/orbti-framework/references/refine-format.md
@./.claude/orbti-framework/references/model-routing.md
</execution_context>

<context>
$ARGUMENTS

@.orbti/PROJECT.md
@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/refine.md
</process>

<success_criteria>
- [ ] REFINE.md created in correct project directory
- [ ] All acceptance criteria defined
- [ ] STATE.md updated with loop position
</success_criteria>
