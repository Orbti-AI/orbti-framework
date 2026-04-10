---
name: orbti:progress
description: Smart status with routing - suggests ONE next action
argument-hint: "[context]"
allowed-tools: [Read]
---

<objective>
Show current progress and **route to exactly ONE next action**. Prevents decision fatigue by suggesting a single best path.

**When to use:**
- Mid-session check on progress
- After `/orbti:resume` for more context
- When unsure what to do next
- To get a tailored suggestion based on your current focus
</objective>

<model>haiku</model>

<execution_context>
</execution_context>

<context>
$ARGUMENTS

@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>

<step name="load_state">
Read `.orbti/STATE.md` and `.orbti/ROADMAP.md`.

**Projeto ativo:** identificar qual projeto está em foco (do STATE.md → Loop Position).

**Se há LOOP.md no projeto ativo**, ler também:
```bash
cat .orbti/projects/{projeto-ativo}/LOOP.md
```

Extrair de STATE.md:
- Projeto ativo e loop atual
- Loop position (REFINE/BUILD/INTEGRATE markers)

Extrair de LOOP.md (se existir):
- Total de loops
- Por loop: lista de refines, task count, status de cada um
- Loops completas vs pendentes
</step>

<step name="calculate_progress">
**Se LOOP.md existe:**

Calcular por loop:
- Loops completas (todos INTEGRATE ✅): contar
- Loop atual: qual está em andamento, quantos REFINEs, task count total
- Loops pendentes: quantas ainda faltam, task count estimado

Calcular progresso percentual da loop atual:
- Tasks totais na loop = soma de tasks de todos os REFINEs da loop
- Tasks concluídas = tasks dos REFINEs já integrados na loop
- % = concluídas / totais

**Se LOOP.md não existe:**
- Progresso baseado em REFINE/BUILD/INTEGRATE do STATE.md
</step>

<step name="consider_user_context">
**If `[context]` argument provided:**

User has given additional context about their current focus or constraint.
Factor this into routing decision:
- "I need to fix a bug first" → prioritize that over planned work
- "I only have 30 minutes" → suggest smaller scope
- "I want to finish this phase" → stay on current path
- "I'm stuck on X" → suggest debug or research approach

**If no argument:** Use default routing based on state alone.
</step>

<step name="determine_routing">
Based on state (+ user context if provided), determine **ONE** next action:

**Default routing (no user context):**

| Situation | Single Suggestion |
|-----------|-------------------|
| No refine exists | `/orbti:refine` |
| Refine awaiting approval | "Approve refine to proceed" |
| Refine approved, not executed | `/orbti:build [path]` |
| Applied, not unified | `/orbti:integrate [path]` |
| Loop complete, more phases | `/orbti:refine` (next phase) |
| Milestone complete | "Create next milestone or ship" |
| Blockers present | "Address blocker: [specific]" |
| Context at DEEP/CRITICAL | `/orbti:pause` |

**With user context:** Adjust suggestion to align with stated intent.

**IMPORTANT:** Suggest exactly ONE action. Not multiple options.
</step>

<step name="display_progress">
Show progress with single routing.

**Se LOOP.md existe para o projeto ativo**, mostrar breakdown por loop:

```
════════════════════════════════════════
ORBTI PROGRESS — {projeto}
════════════════════════════════════════

Loops: {X} de {Y} completas

Loop 1 — {nome}          ✅  [{N} tasks, {M} refines]
Loop 2 — {nome}          ⏳  [{N} tasks, {M} refines — BUILD done]
  ├── 02-REFINE-FRONT  ⏳  1 task — built
  └── 03-REFINE-FRONT  ⏳  1 task — built
Loop 3 — {nome}          🔲  [{N} tasks, {M} refines — pendente]

Loop atual — Loop 2:
┌─────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE     │
│    ✓        ✓        ○             │
└─────────────────────────────────────┘

────────────────────────────────────────
▶ NEXT: /orbti:integrate .orbti/projects/{projeto}
  Fecha loop 2, libera loop 3.
────────────────────────────────────────
```

**Se LOOP.md não existe** (projeto sem múltiplas loops):

```
════════════════════════════════════════
ORBTI PROGRESS
════════════════════════════════════════

{projeto}: [{refine atual}]
┌─────────────────────────────────────┐
│  REFINE ──▶ BUILD ──▶ INTEGRATE     │
│    ✓        ✓        ○             │
└─────────────────────────────────────┘

────────────────────────────────────────
▶ NEXT: {comando único}
────────────────────────────────────────
```

Type "yes" to proceed, or provide context for a different suggestion.
</step>

<step name="context_advisory">
If context is at DEEP or CRITICAL bracket:

```
⚠️ Context Advisory: Session at [X]% capacity.
   Recommended: /orbti:pause before continuing.
```
</step>

</process>

<success_criteria>
- [ ] Overall progress displayed visually
- [ ] Current loop position shown
- [ ] Exactly ONE next action suggested (not multiple)
- [ ] User context considered if provided
- [ ] Context advisory shown if needed
- [ ] Se LOOP.md existe: breakdown por loop com task count e status de cada refine
</success_criteria>

<workflow-dev>
# Decisões de Design — PROGRESS command

## Breakdown por Loop (2026-04-08)

**Decisão:** Quando o projeto ativo tem LOOP.md, o progress mostra status por loop com contagem de tasks e refines, não apenas o loop REFINE/BUILD/INTEGRATE genérico.

**Rationale:** Com múltiplas loops, o loop loop simples (✓/✓/○) não diz quanto falta. O usuário precisa saber: quantas loops existem, qual está em andamento, quantas tasks ainda restam, qual o próximo bloco a ser construído. LOOP.md é a fonte dessas informações.

**Formato de exibição quando LOOP.md existe:**
```
Loop 1 — {nome}   ✅  [N tasks, M refines]
Loop 2 — {nome}   ⏳  [N tasks, M refines — BUILD done]
  ├── 02-REFINE-FRONT  ⏳  1 task
  └── 03-REFINE-FRONT  ⏳  1 task
Loop 3 — {nome}   🔲  [N tasks, M refines — pendente]
```

**Cálculo de progresso:**
- Task count vem do LOOP.md (contagem definida no REFINE durante o planning)
- Status por refine: sem INTEGRATE = ⏳ ou 🔲, com INTEGRATE = ✅
- Percentual da loop = tasks dos refines integrados / tasks totais da loop
</workflow-dev>
