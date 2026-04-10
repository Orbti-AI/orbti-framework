<purpose>
Reconcile what was planned vs. what was built. Create one INTEGRATE.md per loop (covering all specialists), update STATE.md, and close the loop to prepare for the next.

**Regra fundamental:** INTEGRATE é por loop, não por specialist. Só executa quando TODOS os specialists do loop (F, B, T, A) encerraram o BUILD. Loop parcial → apenas pause.
</purpose>

<when_to_use>
- Todos os specialists do loop concluíram BUILD
- Pronto para fechar o loop inteiro e preparar o próximo
- NUNCA executar INTEGRATE com loop parcialmente construído
</when_to_use>

<loop_context>
Expected phase: INTEGRATE
Prior phase: BUILD (execution complete)
Next phase: REFINE (next refine or next phase)
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/projects/{project}/{refine}-REFINE.md
@.orbti/napkin.md (if exists — read silently, use to reconstruct session context)
</required_reading>

<references>
@./.claude/orbti-framework/references/loop-phases.md
@./.claude/orbti-framework/templates/INTEGRATE.md
@./.claude/orbti-framework/workflows/transition-phase.md (loaded when last refine in phase)
</references>

<process>

<step name="check_loop_complete" priority="blocking">
**PRIMEIRO: verificar se o loop inteiro está pronto para integrar.**

## Identificar loop atual

```bash
# Detectar loop pelo argumento ou pelo STATE.md
LOOP_NUM=$(grep "^loop:" .orbti/projects/{slug}/{refine}-REFINE.md | cut -d: -f2 | tr -d ' ')
```

## Listar todos os specialists do loop

```bash
# Todos os REFINEs do loop
ls .orbti/projects/{slug}/*-loop${LOOP_NUM}-*-REFINE.md 2>/dev/null
```

## Verificar quais specialists concluíram BUILD

Para cada REFINE do loop, verificar se há evidência de BUILD completo. Um specialist concluiu BUILD quando:
- Tem um `NN-loopN-S-BUILD.md` ou equivalente de log de execução, **OU**
- O usuário confirmou explicitamente que o BUILD do specialist foi concluído

```bash
# Checar REFINEs sem BUILD completo documentado
for f in .orbti/projects/{slug}/*-loop${LOOP_NUM}-*-REFINE.md; do
  specialist=$(echo $f | grep -o '\-[FBTA]\-REFINE' | tr -d '-REFINE')
  echo "Specialist $specialist: verificar BUILD"
done
```

---

### SE loop incompleto (algum specialist ainda em BUILD):

```
════════════════════════════════════════
⏸ INTEGRATE INDISPONÍVEL — LOOP PARCIAL
════════════════════════════════════════
Loop {N} ainda não está completo.

Specialists concluídos:  {lista}
Specialists pendentes:   {lista}

INTEGRATE só executa quando todos os specialists fecharem BUILD.

O que fazer agora?
[1] Pausar — salvar estado e continuar depois
[2] Cancelar
════════════════════════════════════════
```

**Se "Pausar":**
1. Atualizar STATE.md com o progresso parcial:
   ```markdown
   **Loop {N} — em andamento**
   Specialists concluídos: {lista com ✓}
   Specialists pendentes:  {lista com ○}
   Próximo: concluir BUILD de {specialist pendente}
   ```
2. **Não criar INTEGRATE.md** — loop não está fechado.
3. Exibir: `Estado salvo. Loop {N} continua quando os specialists pendentes concluírem BUILD.`
→ PARAR.

---

### SE loop completo (todos os specialists concluíram BUILD):

→ Prosseguir para `test_gate`.
</step>

<step name="test_gate" priority="first">
**INTEGRATE só roda após testes aprovados. Verificar evidência de testes antes de prosseguir.**

```bash
ls .orbti/projects/{projeto}/{refine}-TEST-PLAN.md 2>/dev/null || echo "NOT_FOUND"
```

**Se TEST-PLAN.md não existe:**
```
════════════════════════════════════════
⛔ INTEGRATE BLOQUEADO
════════════════════════════════════════
Não há evidência de testes para este refine.
INTEGRATE só pode rodar após /orbti:test executar e aprovar os cenários.

Execute primeiro:
  /orbti:test .orbti/projects/{projeto}/{refine}-REFINE.md

Após testes aprovados, rode INTEGRATE novamente.
════════════════════════════════════════
```
→ PARAR. Não prosseguir.

**Se TEST-PLAN.md existe mas status = `pending` ou `running`:**
```
════════════════════════════════════════
⛔ INTEGRATE BLOQUEADO
════════════════════════════════════════
TEST-PLAN existe mas testes não foram concluídos (status: {status}).
Aguarde /orbti:test completar antes de prosseguir.
════════════════════════════════════════
```
→ PARAR.

**Se TEST-PLAN.md existe com status = `failed`:**
```
════════════════════════════════════════
⚠ INTEGRATE COM FALHAS DE TESTE
════════════════════════════════════════
TEST-PLAN reporta falhas. Você pode:
[1] Prosseguir mesmo assim (falhas documentadas no INTEGRATE.md)
[2] Corrigir antes — rodar /orbti:refine-fix

Falhas encontradas: {lista do TEST-PLAN}
════════════════════════════════════════
```
→ Aguardar decisão do usuário. Se [1]: prosseguir com nota de falhas.

**Se TEST-PLAN.md existe com status = `passed` ou `partial`:**
→ Prosseguir normalmente com os passos abaixo.
→ Incluir referência ao TEST-PLAN no INTEGRATE.md.
</step>

<step name="gather_results">
1. Read `.orbti/napkin.md` if it exists:
   - Find the most recent session entry matching this project/refine
   - Extract: what was done, corrections made, surprises, what worked well
   - This is your primary source of truth for what happened in BUILD
   - Apply silently — do not announce that you read it

2. Recall execution from BUILD phase (supplement with napkin):
   - Which tasks completed successfully
   - Which tasks failed (if any)
   - Which checkpoints were resolved and how
   - Any deviations from the refine

3. Read REFINE.md to refresh:
   - Original acceptance criteria
   - Expected outputs
   - Task definitions

4. If napkin has `Corrections & Redirects` entries for this session:
   - Include them automatically in the Deviations section of INTEGRATE.md
   - These are already documented — don't lose them
</step>

<step name="compare_refine_vs_actual">
1. For each acceptance criterion:
   - Was it satisfied? (PASS/FAIL)
   - Evidence of satisfaction
2. For each task:
   - Did it complete as specified?
   - Any modifications to approach?
3. Note deviations:
   - What differed from refine
   - Why it differed
   - Impact on outcomes
</step>

<step name="audit_skill_invocations">
**Check specialized workflow usage (if configured):**

1. Check if .orbti/SPECIAL-FLOWS.md exists:
   ```bash
   ls .orbti/SPECIAL-FLOWS.md 2>/dev/null
   ```

2. If not exists: Skip this step entirely

3. If exists:
   a. Read SPECIAL-FLOWS.md
   b. For each skill with priority "required":
      - Check if skill was invoked during this BUILD phase
      - Mark as ✓ (invoked) or ○ (gap)
   c. For each required skill — check if it has `auto_invoke: true` in SPECIAL-FLOWS.md:
      - If `auto_invoke: true` AND skill not yet invoked in this session:
        - Read the `invoke_instructions` field from SPECIAL-FLOWS.md for that skill
        - Execute the instructions exactly as declared — no user confirmation needed
        - Mark as ✓ after execution
      - If `auto_invoke` absent or false AND skill not invoked:
        - Add to STATE.md Deviations section:
          ```markdown
          ### Skill Audit (Phase [N])
          | Expected | Invoked | Notes |
          |----------|---------|-------|
          | /skill-name | ○ | [reason if known] |
          ```
        - Warn user: "Skill gap(s) documented in STATE.md. Review before next phase."
   d. Do NOT block INTEGRATE for skill gaps (warn only)

4. If all required skills invoked:
   - Note in INTEGRATE.md: "Skill audit: All required skills invoked ✓"

**Reference:** @references/specialized-workflow-integration.md
</step>

<step name="structured_code_review" priority="required">
**Code review estruturado — executar sempre antes de criar o INTEGRATE.md.**

## Fonte do review (em ordem de preferência)

1. **Skill code-reviewer disponível:**
   ```bash
   ls .claude/skills/code-reviewer* 2>/dev/null || \
   grep "code-reviewer\|code-review" .claude/settings.json 2>/dev/null
   ```
   Se disponível → invocar `/skill code-reviewer` com os arquivos modificados.

2. **superpowers:code-reviewer agent:**
   Se skill não disponível → usar o agente especializado `code-reviewer` via Agent tool.

3. **Review inline estruturado:**
   Se nenhum acima → executar review diretamente com a estrutura abaixo.

## O que o review avalia

| Lens | O que verifica | Severidades |
|------|---------------|-------------|
| Qualidade | Clareza, consistência, padrões do projeto | minor / major / blocker |
| Segurança | Inputs, exposição, injeção, auth, secrets | minor / major / blocker |
| Cobertura de ACs | Cada AC do REFINE.md foi satisfeita? | pass / fail |
| Escopo do refine | Mudanças fora do REFINE.md? | minor / blocker |

## Decisão por severidade

- **blocker** → PARAR. Não fechar loop. Abrir `/orbti:refine-fix`.
- **major** → Incluir no INTEGRATE.md. Perguntar ao usuário se prossegue.
- **minor** → Registrar como deferred issue no INTEGRATE.md. Não bloqueia.

## Criar arquivo de review

Salvar em `.orbti/projects/{slug}/NN-loopN-S-REVIEW.md`:

```markdown
# Code Review — {slug} Loop N ({Specialist})

**Resultado:** PASS | PASS COM RESSALVAS | FAIL
**Data:** YYYY-MM-DD

## Qualidade
- [Achado] — [minor | major | blocker]

## Segurança
- [Achado] — [minor | major | blocker]

## Cobertura de ACs
| AC | Descrição | Status |
|----|-----------|--------|
| 1  | [descrição] | ✓ PASS |
| 2  | [descrição] | ✗ FAIL |

## Escopo
- Mudanças fora do REFINE: [nenhuma | lista]

## Itens Diferidos
- [minor finding para resolver depois]
```

**Nota sobre git:** O integrate NÃO faz git commit ou push. Isso é responsabilidade de special flows separados (`commit`, `commit-push`, `commit-push-pr`). O integrate apenas documenta e fecha o loop.
</step>

<step name="review_team_optional">
**Revisão em equipe (opcional — se agent teams habilitados):**

Check if agent teams are enabled:
```bash
grep "agent_teams:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Teams are **off by default** — only active if `agent_teams.enabled: true` in `.orbti/config.md`.

**If teams enabled:** complementar o review estruturado com 3 reviewers paralelos (code quality, security, AC coverage). Integrar findings ao REVIEW.md já criado.

**If teams not available:** o `structured_code_review` step acima já cobre completamente. Prosseguir para `create_summary`.
</step>

<step name="create_summary">
**INTEGRATE.md é um por loop — cobre todos os specialists.**

1. Criar em `.orbti/projects/{slug}/loopN-INTEGRATE.md`

   **Frontmatter:**
   ```yaml
   ---
   project: {slug}
   loop: N
   specialists: [F, B, T, A]   # lista dos specialists do loop
   completed: ISO timestamp
   ---
   ```

   **Sections:**
   - Loop Objective (objetivo do loop — do COCREATE ou REFINE)
   - Specialists do Loop (tabela: specialist, refine, resultado)
   - What Was Built (tabela consolidada: arquivo, specialist, propósito)
   - Acceptance Criteria Results (tabela consolidada de todos os ACs do loop)
   - Code Reviews (links para cada `NN-loopN-S-REVIEW.md` + resultado)
   - Deviations (desvios de qualquer specialist, com explicações)
   - Deferred Issues (findings menores — endereçar no próximo loop ou backlog)
   - Next Loop (o que vem a seguir)

   **Nota:** git commit/push NÃO são executados aqui. São special flows separados.
</step>

<step name="update_runbook">
**Update agent learning from this BUILD:**

1. Check if `.orbti/RUNBOOK.md` exists
2. If exists: curate — re-prioritize by impact, merge duplicates, remove stale entries
3. If not exists AND this BUILD had corrections, deviations, or notable patterns: create it
4. Add any entries logged during `execute_tasks` that weren't yet written
5. Rules:
   - Keep each category ≤ 10 items, highest impact first
   - Every entry must have a `Do instead:` line
   - Remove entries that are now obvious or no longer apply
   - Do NOT log one-off events or routine completions

Skip if BUILD was clean with no notable corrections or surprises.
</step>

<step name="update_state">
Update STATE.md — three sections:

**1. Projects Overview table:**
- Find the current project row
- Update loop position for this refine: `✓ ✓ ◉` → `✓ ✓ ✓`
- Increment loop count: `N/M` → `N+1/M`
- If more loops remain: keep Status as `🔵 In Progress`
- If last loop: Status → `✅ Complete` (transition-phase handles this)

**2. Current Focus:**
```
**Project:** [N] — [Name]
**Refine:** [A] complete
**Status:** Ready for next REFINE (or transitioning)
**Last activity:** [timestamp] — Refine [A] integrated

Loop position:
REFINE ──▶ BUILD ──▶ INTEGRATE
  ✓        ✓        ✓     [Loop complete - ready for next REFINE]
```

**3. Session Continuity:**
- Stopped at: Refine [A] integrated
- Next action: `/orbti:refine` for Refine [A+1] (or next project)
- Resume file: point to the INTEGRATE.md just created
</step>

<step name="check_phase_completion">
**Determine if this is the last refine in the phase:**

1. Count REFINE.md files in current project directory
2. Count INTEGRATE.md files (including the one just created)
3. Compare counts:
   - If REFINE count = SUMMARY count → Last refine, trigger transition
   - If REFINE count > SUMMARY count → More loops remain in project
</step>

<step name="route_based_on_completion">
**If more refines remain in phase:**

Report with quick continuation:
```
════════════════════════════════════════
REFINE COMPLETE
════════════════════════════════════════

Refine: {refine} — [description]
[summary of what was built]
[deviations if any]

Project {N} progress: {X}/{Y} loops complete

---
Continue to next loop?

[1] Yes, refine {NN+1} | [2] Pause here
════════════════════════════════════════
```

**Accept:** "1", "yes", "continue" → run `/orbti:refine` for next loop in same project
</step>

<step name="check_worktree_completion">
**Se este INTEGRATE está rodando dentro de um loop worktree (`worktree_branch` no REFINE.md):**

1. Verificar REFINE.md frontmatter:
   ```bash
   grep "worktree_branch:" .orbti/projects/{project}/{refine}-REFINE.md
   ```

2. Se `worktree_branch` presente → **este é o passo final do loop isolado.**

3. Reportar ao usuário para revisão — **não mostrar opções ainda**:
   ```
   ════════════════════════════════════════
   LOOP COMPLETO — aguardando sua revisão
   ════════════════════════════════════════
   Build → Test → Integrate: concluídos
   Branch:   [worktree_branch]
   Worktree: [worktree_path]

   Revise antes de decidir:

   Código alterado:
     cd [worktree_path] && git diff main

   Vídeos E2E:
     [worktree_path]/tests/e2e/videos/

   Quando terminar a revisão, digite "aprovado" para
   ver as opções de merge, ou "descartar" para cancelar.
   ════════════════════════════════════════
   ```

4. Aguardar input do usuário:
   - "aprovado" / "aprovar" / "ok" / "merge" → mostrar opções:
     ```
     O que fazer com as mudanças?

     [1] Merge na branch atual
         git merge [worktree_branch]

     [2] Commitar nesta branch (você decide push depois)
         git commit dentro da worktree

     [3] Descartar tudo
         git worktree remove [worktree_path]
         git branch -d [worktree_branch]
     ```
   - "descartar" / "cancelar" / "rejeitar" → executar opção 3 diretamente

5. Se `worktree_branch` ausente → pular este step, seguir fluxo normal.
</step>

<step name="execute_transition" priority="required" gate="blocking">
**If last refine in phase — TRANSITION IS MANDATORY:**

⚠️ **NEVER skip this step. This is what makes ORBTI a system, not random loops.**

1. Announce clearly:
   ```
   ════════════════════════════════════════
   PROJECT {N} COMPLETE — TRANSITION REQUIRED
   ════════════════════════════════════════
   ```

2. **MUST read and execute:** @./.claude/orbti-framework/workflows/transition-phase.md

3. Transition handles (do not skip any):
   - Evolve PROJECT.md (requirements validated → shipped)
   - Update ROADMAP.md (phase status → complete)
   - Git commit for phase: `feat({project-name}): {description}`
   - Clean stale handoffs
   - Route to next phase or milestone completion

4. **Only after transition completes** → offer next phase routing

**Anti-pattern:** Closing INTEGRATE and immediately offering `/orbti:refine` for next phase WITHOUT running transition. This breaks system cohesion and skips git commits.
</step>

</process>

<output>
- INTEGRATE.md at `.orbti/projects/{project-name}/{refine}-INTEGRATE.md`
- Updated STATE.md
- Updated ROADMAP.md (if phase complete)
</output>

<error_handling>
**BUILD not complete:**
- Check STATE.md for actual position
- If BUILD incomplete, cannot INTEGRATE
- Return to BUILD to complete or document blockers

**Missing execution context:**
- If no memory of execution results, read any logs
- Ask user to confirm what was completed
- Document uncertainty in INTEGRATE.md

**REFINE.md missing:**
- Cannot reconcile without original refine
- Ask user to locate or reconstruct
</error_handling>

<anti_patterns>
**Skipping SUMMARY:**
Every completed refine MUST have a INTEGRATE.md. No exceptions.

**Partial state updates:**
Update ALL of: SUMMARY, STATE, ROADMAP (if phase done). Don't leave partial.

**Vague summaries:**
"It worked" is not a summary. Document files, AC results, deviations specifically.

**Forgetting loop position:**
Always show the visual loop position in STATE.md.
</anti_patterns>
