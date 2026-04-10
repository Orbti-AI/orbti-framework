<purpose>
Execute an approved REFINE.md by running tasks in order, verifying each, handling checkpoints, and recording results for INTEGRATE phase reconciliation.
</purpose>

<when_to_use>
- User has approved a REFINE.md (explicit approval required)
- STATE.md shows loop position at REFINE complete, ready for BUILD
- No unresolved blockers from planning project
</when_to_use>

<loop_context>
Expected phase: BUILD
Prior phase: REFINE (approval just received)
Next phase: TEST → INTEGRATE
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/projects/{project}/{refine}-REFINE.md
</required_reading>

<references>
@./.claude/orbti-framework/references/checkpoints.md (if refine has checkpoints)
@./.claude/orbti-framework/references/loop-phases.md
@./.claude/orbti-framework/references/tdd.md (se test_writer.enabled: true)
</references>

<process>

<step name="check_parallel_mode" priority="blocking">
**Primeiro: detectar o modo de invocação pelo argumento passado.**

## Regra de roteamento por argumento

| Argumento | Modo | Comportamento |
|-----------|------|---------------|
| Arquivo REFINE (ex: `.orbti/projects/foo/01-loop1-F-REFINE.md`) | **single** | Executa APENAS aquele refine — explícito e exclusivo |
| Diretório de projeto (ex: `.orbti/projects/foo`) | **next-loop** | Executa a PRÓXIMA loop pendente — apenas ela |
| Nome do projeto (ex: `eficiencia-debito`) | **next-loop** | Resolve para o diretório, comportamento idêntico |

**Se argumento é um arquivo:**
→ Modo **single**: executa apenas aquele refine. Ignora todos os outros. Ir para `validate_approval`.

**Se argumento é um diretório ou nome de projeto:**
→ Modo **next-loop**: identificar a próxima loop pendente, executar SOMENTE ela. Parar ao terminar a loop — loop N+1 requer novo `/orbti:build`.

---

## Identificar próxima loop pendente (modo next-loop)

**1. Ler LOOP.md se existir:**
```bash
cat .orbti/projects/{projeto}/LOOP.md 2>/dev/null
```
LOOP.md é a fonte de verdade sobre a estrutura de loops do projeto. Se existir, usá-la para identificar qual loop está pendente.

**2. Fallback — escanear REFINEs sem INTEGRATE (novo naming):**
```bash
for f in .orbti/projects/{projeto}/[0-9]*-loop*-*-REFINE.md; do
  base="${f%-REFINE.md}"
  [ ! -f "${base}-INTEGRATE.md" ] && echo "$f"
done
```
Agrupar por `loop:` do frontmatter. O menor número de loop sem todos os REFINEs integrados = próxima loop.

**Nota sobre refine T (Test):** ao identificar refines da loop, incluir o `NN-loopN-T-REFINE.md` se existir — ele faz parte da loop e deve ser executado junto.

**3. Determinar modo de execução:**

```bash
grep "parallel_build:" .orbti/config.md 2>/dev/null -A1 | grep "enabled: true"
echo $CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS
```

Se `parallel_build.enabled: false` OU env var ausente → sequencial:
```
⚠ Rodando sequencial (parallel_build desativado ou CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS não setado).
Para ativar paralelo: ./.claude/settings.json → { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

Se `parallel_build.enabled: true` e env var = "1":
- 1 REFINE na loop → modo single
- 2+ REFINEs na loop → `parallel_team_build`
</step>

<step name="check_test_refine">
**Verificar se há refine T (Test) na loop sendo executada.**

```bash
# Detectar refine T da loop atual
LOOP_NUM=$(grep "^loop:" {refine_path} | cut -d: -f2 | tr -d ' ')
ls .orbti/projects/{projeto}/[0-9]*-loop${LOOP_NUM}-T-REFINE.md 2>/dev/null
```

**Se refine T existe:**
- Incluir na execução da loop (junto com F, B, A)
- O BUILD irá construir os testes automatizados definidos no refine T
- Exibir ao usuário: `✓ Refine T encontrado — testes automatizados serão construídos`

**Se refine T não existe:**
- Não construir testes automatizados
- Silencioso — não avisar ao usuário sobre ausência de testes
- O TEST workflow (manual/checkpoint) ainda pode ser executado após o BUILD
</step>

<step name="check_tdd_mode" priority="first">
**Verificar se o projeto usa TDD (testes escritos antes do BUILD):**

```bash
grep -i "tdd\|test.first\|test-first" .orbti/SPECIAL-FLOWS.md 2>/dev/null
```

**Se TDD declarado em SPECIAL-FLOWS.md:**

1. Verificar se já existem testes escritos para este refine:
   ```bash
   ls testes/*/$(basename $REFINE_PATH | sed 's/-REFINE.md//')/ 2>/dev/null || \
   cat .orbti/TESTS.md 2>/dev/null | grep -A2 "path:" | head -6
   ```

2. Se testes NÃO existem ainda → **BLOQUEAR BUILD**:
   ```
   ════════════════════════════════════════
   ⛔ BLOQUEADO: Projeto em modo TDD
   ════════════════════════════════════════
   Testes devem ser escritos ANTES do BUILD.

   ▶ NEXT: /orbti:test [refine-path]
     Escreva os testes (devem falhar), depois aprove o BUILD.
   ════════════════════════════════════════
   ```

3. Se testes EXISTEM → confirmar que estão falhando (esperado em TDD):
   ```
   ✓ Testes encontrados — modo TDD ativo.
   Os testes devem estar falhando agora (red).
   O BUILD vai fazê-los passar (green).
   ```
   → Continuar para `validate_approval`.

**Se TDD não declarado:** → continuar para `validate_approval` normalmente.
</step>

<step name="validate_approval" priority="first">
1. Confirm user has explicitly approved the refine
   - Do NOT assume approval
   - Look for explicit signal: "approved", "execute", "go ahead", "background", etc.
2. Read STATE.md to verify:
   - Loop position shows REFINE complete
   - Correct phase and refine identified
3. If approval unclear:
   - Ask: "Refine ready at [path]. Approve refine execution?"
   - Wait for explicit approval before proceeding
4. Proceed with foreground execution.
</step>

<step name="background_build">
**Background mode — only available for refines with `autonomous: true`.**

Check REFINE.md frontmatter:
```
autonomous: true   → executa todas as tasks direto, sem pedir confirmação a cada passo
autonomous: false  → pausa após cada task para revisão/confirmação do usuário — não pode rodar em background
```

**Com `autonomous: true`, o agente ainda para em:**
- `checkpoint:human-verify` — validação humana explícita no REFINE
- `checkpoint:human-action` — ação humana indispensável
- Cenários `manual` no TEST-PLAN
- INTEGRATE — **sempre** requer aprovação humana

Se `autonomous: false`:
```
Este refine requer confirmação a cada task — não pode rodar em background.
Rodar em foreground? [sim / não]
```

**Worktree: isolamento nativo via parâmetro `isolation: "worktree"` do Agent tool:**

Se `worktree.enabled: true` no frontmatter do REFINE.md:

Passar `isolation: "worktree"` diretamente no Agent tool ao spawnar o builder:

```
Agent(
  name: "builder-{slug}",
  run_in_background: true,
  isolation: "worktree",     ← Claude Code cria branch física isolada automaticamente
  prompt: "..."
)
```

O Claude Code gerencia o worktree automaticamente:
- Cria um branch isolado fisicamente separado
- O agente trabalha nessa cópia sem afetar a branch atual
- Se o agente fez mudanças: `worktree_path` e `branch` são retornados no resultado
- Se o agente não fez mudanças: worktree é removido automaticamente

No BUILD COMPLETE (quando `worktree_path` e `branch` retornados):
```
Branch: [branch retornada]
Path:   [worktree_path retornado]

Para revisar:  cd [worktree_path] && git diff main
```

No INTEGRATE: usar branch retornada para merge.

Se `worktree: enabled: false` ou ausente → spawnar normalmente sem `isolation`.

Se `autonomous: true`, spawn a background agent:

```
Spawn background agent for: [refine-path]

O agente deve:
1. Executar todas as tasks em ordem
2. Rodar cada <verify>
3. Respeitar todos os <boundaries>
4. Se test_writer.enabled: seguir TDD para cada task (RED→GREEN→REFACTOR)
5. Ao encontrar checkpoint:human-verify ou human-action: PARAR, reportar ao lead via SendMessage
6. Ao encontrar cenário `manual` no TEST-PLAN: registrar como 'skipped' e continuar
7. Em falha de task: logar, parar, NÃO continuar para próxima task
8. Ao concluir BUILD + TESTS: reportar 'BUILD DONE: <refine-id> | TESTS: <status>'
9. NUNCA rodar INTEGRATE — parar após BUILD DONE
```

Confirm to user:
```
════════════════════════════════════════
BUILD RUNNING IN BACKGROUND (worktree isolado)
════════════════════════════════════════

Refine: [refine-path]
Tasks: [N] tasks

Execução isolada — sua branch atual não será afetada.
Você será notificado quando concluir.
════════════════════════════════════════
```

When background agent completes:

**Se o agente fez mudanças** (worktree_path e branch retornados):
```
════════════════════════════════════════
BUILD COMPLETE — worktree disponível
════════════════════════════════════════
[execution summary]
────────────────────────────────────────
Branch: [branch retornada pelo agente]
Path:   [worktree_path retornado pelo agente]

Para revisar:  cd [worktree_path] && git diff main
Para aprovar:  git merge [branch]
Para descartar: git worktree remove [worktree_path] && git branch -d [branch]
────────────────────────────────────────
Continue to TEST?
[1] Yes, run /orbti:test | [2] Pause here
════════════════════════════════════════
```

**Se o agente não fez mudanças** (worktree removida automaticamente):
Apresentar o BUILD COMPLETE summary padrão do foreground finalize step, então oferecer INTEGRATE.
</step>

<step name="load_plan">
1. Read the REFINE.md file
2. Parse frontmatter:
   - type: determines execution route (execute | prototype | tdd | research)
   - autonomous: determines checkpoint handling
   - files_modified: track what we'll change
   - depends_on: verify dependencies met
3. **Se `type: front`:** ir para `front_build` — não executar fluxo backend padrão.
4. Extract tasks from <tasks> section
5. Note boundaries from <boundaries> section
6. Load acceptance criteria for verification reference
</step>

<step name="front_build">
**Executar quando `type: front`. Constrói frontend — componentes React, páginas, UI.**

## 1. Detectar contexto de execução

Antes de construir, verificar se este REFINE-FRONT declara um backend associado:

```bash
# 1. Checar campo integrates_with no frontmatter (declaração explícita — preferencial)
grep "^integrates_with:" {refine-path}

# 2. Fallback: escanear backends na mesma loop sem INTEGRATE.md
for f in .orbti/projects/{projeto}/[0-9]*-REFINE.md; do
  [ ! -f "${f%-REFINE.md}-INTEGRATE.md" ] && grep -q "type: execute\|type: tdd" "$f" 2>/dev/null && echo "$f"
done
```

**Preferir `integrates_with` quando presente** — é a declaração explícita do refine. O scan é fallback para REFINEs criados antes desta feature.

**Dois cenários possíveis:**

### Cenário A — Só frontend (sem backend na loop)

→ Construir com dados mockados. Finalizar normalmente. Ir para `finalize`.

### Cenário B — Frontend + Backend em paralelo

→ Executar em duas fases:

**Fase 1 — Construir com mocks (não aguardar backend)**

Construir o frontend completo usando dados mockados:
- Componentes, páginas, layout
- Dados mockados inline ou em arquivo `_mock.ts`
- Todos os ACs de UI verificados com dados mockados

Quando fase 1 completa → se rodando como builder em parallel_team_build:
```
SendMessage → lead: "FRONT MOCK DONE: <refine-id> — frontend construído com mocks, aguardando backend"
```

**Fase 2 — Aguardar backend e finalizar integração**

Aguardar o builder do backend reportar `BUILD DONE` (via notificação do lead ou diretamente).

Quando backend disponível, executar tasks de integração:
- Substituir chamadas mockadas pelos hooks/services reais
- Conectar ao endpoint criado pelo backend
- Remover arquivos `_mock.ts` se dados agora vêm da API
- Verificar que a tela funciona com dados reais (ou com os types/contracts exportados pelo backend)

**Se rodando em modo sequencial (sem Agent Teams):**
O lead verifica manualmente se o backend foi construído antes de continuar a fase 2.
Se backend ainda não foi construído → pausar e informar:
```
⏸ Frontend (fase 1 com mocks) concluído.
  Aguardando backend ser construído antes de integrar.
  Quando backend estiver pronto: /orbti:build [projeto] para continuar fase 2.
```

---

## 2. Carregar referências de design

- Ler frontmatter do REFINE-FRONT.md: `prototype` e `prototype_tool`
  - Se `prototype` presente → usar como referência visual primária durante a construção
  - `prototype_tool` indica onde buscar: paper (MCP paper), pencil (MCP pencil), figma (MCP plugin), html (ler arquivo)
- Se `.orbti-design/system.md` existe → carregar tokens, componentes e padrões antes de executar tasks

## 3. Usar o Component Map

Ler a seção `## Component Map` do REFINE-FRONT.md.
Para cada elemento:
- **reutilizar**: importar o componente exato especificado — não criar alternativa
- **estender**: usar o componente base + aplicar o wrapper/variante descrito
- **criar**: criar o componente novo seguindo as props e tokens especificados no refine

## 4. Executar tasks de frontend

Mesma engine que refines tech: checkpoints, qualidade e verificação padrão.
Diferença: não rodar testes de backend, migrations ou scripts de banco.

Ao concluir (fase 1 ou fase 1+2 dependendo do cenário), ir para `finalize`.
</step>

<step name="verify_required_skills" priority="blocking">
**BLOCKING CHECK: Required skills must be loaded before execution.**

1. Check if REFINE.md has a <skills> section
2. If no <skills> section: proceed (no skill requirements)
3. If <skills> section exists:
   a. For each skill marked "required":
      - Check if skill has been invoked in current session
      - If not invoked: add to missing_skills list
   b. If missing_skills is not empty:
      - **BLOCK execution**
      - Display:
        ```
        ════════════════════════════════════════
        ⛔ BLOCKED: Required skills not loaded
        ════════════════════════════════════════

        This refine requires the following skills:

        Missing:
        - /skill-name → Run: /skill-name
        - /skill-name → Run: /skill-name

        Load these skills now, then type "ready" to continue.
        Or type "override" to proceed without (not recommended).
        ════════════════════════════════════════
        ```
      - Wait for user input
      - If "ready": re-check skills, proceed if all loaded
      - If "override":
        - Log deviation to STATE.md Decisions: "Override: Proceeded without required skills [list]"
        - Proceed with warning
   c. If all required skills loaded:
      - Display: "✓ All required skills loaded"
      - Proceed to execute_tasks

**This check runs BEFORE any task execution, ensuring skills are in place.**
</step>

<step name="execute_tasks">
**Ler `autonomous:` do frontmatter do REFINE antes de iniciar as tasks.**

| autonomous | Comportamento |
|------------|---------------|
| `true` | Executa todas as tasks direto, sem parar. Só para em `checkpoint:*` explícitos. |
| `false` | Após cada task `type="auto"`, apresenta o que foi feito e aguarda confirmação antes de avançar para a próxima. |

---

For each <task> in order:

**If type="auto" e autonomous: true:**
1. Log task start: "Task N: [name]"
2. Executar ação
3. Rodar verify
4. Registrar resultado (PASS/FAIL) e continuar imediatamente para próxima task

**If type="auto" e autonomous: false:**
1. Log task start: "Task N: [name]"
2. Executar ação
3. Rodar verify
4. Apresentar resultado:
   ```
   ──────────────────────────────
   Task [N]/[Total]: [nome]
   ✓ [O que foi feito — arquivos criados/modificados]
   Verify: [output do verify]
   ──────────────────────────────
   Continuar para task [N+1]? (sim / abortar)
   ```
5. Aguardar confirmação antes de continuar
6. Se "abortar": registrar onde parou, oferecer retomada

**If type="checkpoint:human-verify":**
1. Stop execution
2. Present checkpoint clearly:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Human Verification
   ════════════════════════════════════════

   Task [N] of [Total]: [name]

   What was built:
   [what-built content]

   How to verify:
   [how-to-verify content]

   [resume-signal content]
   ════════════════════════════════════════
   ```
3. Wait for user response
4. On "approved": continue to next task
5. On issues reported: address issues, re-verify, or note deviation

**If type="checkpoint:decision":**
1. Stop execution
2. Present decision with options:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Decision Required
   ════════════════════════════════════════

   Decision: [decision content]
   Context: [context content]

   Options:
   [option-a]: [name] - Pros: [pros] / Cons: [cons]
   [option-b]: [name] - Pros: [pros] / Cons: [cons]

   [resume-signal content]
   ════════════════════════════════════════
   ```
3. Wait for user selection
4. **Record decision to STATE.md:**
   - Open `.orbti/STATE.md`
   - Find `### Decisions` under `## Accumulated Context`
   - Add row: `| [date]: [Decision summary] | Project [N] | [Impact on work] |`
   - Example: `| 2026-01-28: Install in sandbox for testing | Project 1 | Project created in sandbox/box2-orbti-test |`
5. Continue with chosen direction

**If type="checkpoint:human-action":**
1. Stop execution
2. Present required action:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Human Action Required
   ════════════════════════════════════════

   Action: [action content]

   Instructions:
   [instructions content]

   After completion, I will verify:
   [verification content]

   [resume-signal content]
   ════════════════════════════════════════
   ```
3. Wait for user confirmation
4. Run verification check
5. Continue if verified, report if failed
</step>

<step name="parallel_team_build">
**Modo paralelo — Agent Teams. Só chega aqui com 2+ refines elegíveis e env var ativa.**

## Modelo de execução: Loop = um bloco de construção

**Uma invocação de `/orbti:build` = uma loop. Loop N+1 requer novo `/orbti:build`.**

Uma loop pode conter múltiplos REFINEs. Todos os REFINEs da loop são spawnados em paralelo — a diferença está em como eles se comunicam:

```
LOOP 1
  ├── sem depends_on: []    → spawna e executa de forma independente
  └── depends_on: ['02']   → spawna em paralelo MAS comunica com builder-02 via SendMessage
      ↓ quando TODA a loop terminar + INTEGRATE aprovado → pode iniciar LOOP 2
LOOP 2
  ├── sem depends_on: []    → spawna independente
  └── depends_on: ['04']   → spawna em paralelo, comunica com builder-04
```

**Regra de `depends_on`:**
`depends_on` NÃO significa "esperar X terminar antes de começar". Significa "ambos constroem em paralelo, mas há um ponto de comunicação obrigatório entre eles". O builder dependente envia/recebe sinais do builder do qual depende via SendMessage quando precisar de informação (ex: contrato de API, tipos exportados, URL do endpoint).

---

### 1. Agrupar REFINEs por loop e montar grafo de dependências

```bash
# Para cada REFINE pendente, extrair loop e depends_on do frontmatter
for f in .orbti/projects/{projeto}/[0-9]*-REFINE.md; do
  [ ! -f "${f%-REFINE.md}-INTEGRATE.md" ] && echo "$f"
done
```

Construir estrutura:
```
loop 1:
  01 — depends_on: []        → PARALELO (disponível já)
  02 — depends_on: []        → PARALELO (disponível já)
  03 — depends_on: ['02']    → SEQUENCIAL (aguarda 02 dentro da loop)

loop 2:
  04 — depends_on: []        → PARALELO (disponível quando loop 1 terminar)
  05 — depends_on: ['04']    → SEQUENCIAL (aguarda 04 dentro da loop 2)
```

---

### 2. Executar loop por loop

**Para cada loop (em ordem crescente):**

#### 2a. Spawnar todos os REFINEs da loop atual

**Todos os REFINEs da loop são spawnados simultaneamente** — independente de `depends_on`.

`depends_on` não bloqueia o spawn. Controla a **comunicação** entre os builders durante a execução.

Spawnar todos de uma vez em uma única mensagem (paralelo real):

```
Agent tool — uma chamada por REFINE, TODAS na mesma mensagem:

Agent(
  name: "builder-<refine-id>",
  run_in_background: true,                    ← fork — não bloqueia o lead
  isolation: "worktree" (se worktree: enabled: true no REFINE),  ← isolamento nativo
  prompt: "Você é o builder do refine <refine-id>.
           Execute o REFINE.md em: <caminho>
           Siga as regras do workflow build.md (execute_tasks, verify, boundaries).

           ## Testes (se test_writer.enabled: true no config)

           Seguir o ciclo TDD para cada task de implementação:

           **LEI ABSOLUTA: NENHUM CÓDIGO DE PRODUÇÃO SEM UM TESTE FALHANDO PRIMEIRO.**
           Escrever código antes do teste? Deletar. Começar de novo.

           Entrada: TEST-PLAN.md em <caminho-test-plan> — cenários por AC.

           Para cada AC no TEST-PLAN:
           1. RED: Escrever o teste mínimo que descreve o comportamento esperado.
              Rodar. CONFIRMAR que FALHA pelo motivo correto (feature ausente, não typo).
              Se o teste passar: você está testando comportamento já existente — corrigir o teste.
           2. GREEN: Escrever o código MÍNIMO para o teste passar.
              Sem otimizações, sem features extras. Só fazer o teste verde.
              Rodar. CONFIRMAR que passa. Confirmar que outros testes não quebraram.
           3. REFACTOR: Limpar sem adicionar comportamento.
              Rodar. CONFIRMAR que continua verde.

           Racionalizações proibidas:
           - "Vou escrever os testes depois" → deletar o código, começar com TDD
           - "É simples demais para testar" → testes simples levam 30 segundos
           - "Já testei manualmente" → manual não é automático, não reentra
           - "Manter como referência enquanto escrevo os testes" → deletar

           Se agent_teams.enabled: true → após escrever todos os testes (RED completo),
           spawnar agentes em paralelo (um por AC) para GREEN+REFACTOR simultâneos.
           Cada agente reporta: 'TEST PASS: AC-N' ou 'TEST FAIL: AC-N — [erro]'

           Ao finalizar: atualizar TEST-PLAN.md (resultado por cenário + status: passed|failed|partial).

           ## Checkpoints
           Quando encontrar checkpoint (human-verify, decision, human-action):
             - PARE a execução
             - Envie mensagem ao lead via SendMessage: 'CHECKPOINT [tipo] em task [N]: [conteúdo]'
             - Aguarde resposta do lead antes de continuar.

           ## Comunicação com outros builders (se depends_on presente)

           Se este REFINE tem depends_on: ['<outro-id>']:
           - Você e o outro builder estão rodando em paralelo
           - Quando precisar de informação do outro (contrato de API, tipos, endpoint):
             SendMessage → builder-<outro-id>: 'PRECISO: <o que precisa>'
           - Quando o outro te pedir informação:
             Responder via SendMessage com o que foi solicitado
           - Quando tiver algo pronto que o outro precisa (ex: endpoint, types):
             SendMessage → builder-<outro-id>: 'PRONTO: <o que está disponível>'
           - Nunca bloquear indefinidamente — se o outro não responder em tempo razoável,
             continuar com mocks e reportar ao lead

           Ao finalizar todas as tasks: envie ao lead via SendMessage:
           'BUILD DONE: <refine-id> | TESTS: passed/failed/partial/skipped | URLS: <lista de rotas criadas, separadas por vírgula — só se type: front>'

           ⚠ NUNCA rodar INTEGRATE automaticamente.
           INTEGRATE só ocorre quando o usuário aprovar explicitamente.
           Sua responsabilidade termina em BUILD DONE."
)
```

Lançar **todos os REFINEs da loop em uma única mensagem** para execução simultânea real.

Anunciar ao usuário:
```
▶ Loop [N] iniciada:
  [builder-01] → fork background
  [builder-02] → fork background
  [builder-03] → fork background (comunica com builder-02 via depends_on)
```

#### 2b. Monitorar execução da loop

O lead aguarda notificações dos builders em background.

**Quando receber `CHECKPOINT` via SendMessage de um builder:**
- Apresentar ao usuário (human-verify / decision / human-action)
- Aguardar resposta do usuário
- Repassar ao builder via `SendMessage → builder-<refine-id>`
- Builder retoma

**Quando receber `BUILD DONE: <refine-id>` via SendMessage:**
- Registrar conclusão
- Verificar se algum REFINE bloqueado na loop foi desbloqueado por esta conclusão:
  - Se sim → spawnar agora com `run_in_background: true`:
    ```
    ▶ builder-<refine-bloqueado> desbloqueado — fork background iniciado
    ```
  - Se não → aguardar demais builders da loop

#### 2c. Fechar o ciclo da loop antes de avançar

**Loop N+1 só começa após o ciclo completo da loop N: BUILD + INTEGRATE.**

⚠ **INTEGRATE NUNCA ocorre automaticamente — nem para `autonomous: true`.**
O INTEGRATE sempre requer aprovação explícita do usuário, independente do tipo de REFINE.

**Comportamento por tipo de REFINE:**

`autonomous: true` → o builder faz BUILD sem checkpoints e reporta `BUILD DONE`.
O lead aguarda todos os builders reportarem `BUILD DONE`, apresenta o resumo ao usuário e aguarda aprovação para INTEGRATE.

`autonomous: false` → o builder faz BUILD, pausa em checkpoints, lead trata manualmente.
Após BUILD concluído, mesma regra: lead apresenta ao usuário e aguarda aprovação para INTEGRATE.

**Finalizar a loop atual e parar:**

Quando todos os builders da loop atual reportarem `BUILD DONE`:

```
════════════════════════════════════════
LOOP [N] COMPLETA
════════════════════════════════════════
builder-01 → ✓ BUILD DONE
builder-02 → ✓ BUILD DONE  http://localhost:3010/carteira/eficiencia/remessas
builder-03 → ✓ BUILD DONE

Próximo passo obrigatório: TESTE — INTEGRATE só roda após testes aprovados.
Execute: /orbti:test .orbti/projects/[projeto]/
Após testes aprovados: /orbti:integrate [projeto]
Após INTEGRATE: /orbti:build [projeto] para iniciar loop [N+1]
════════════════════════════════════════
```

**PARAR.** Não iniciar loop N+1 automaticamente.
O usuário decide quando rodar `/orbti:build` novamente para a próxima loop.

**Regra absoluta: uma invocação de `/orbti:build` = uma loop. Loop N+1 requer novo `/orbti:build`.**

---

### 3. Finalizar após última loop

Após todos os builders de todas as loops reportarem `BUILD DONE`:

```
════════════════════════════════════════
BUILD COMPLETO
════════════════════════════════════════

Loop 1:
  builder-01 → ✓ concluído
  builder-02 → ✓ concluído
  builder-03 → ✓ concluído (sequencial após 02)

Loop 2:
  builder-04 → ✓ concluído
────────────────────────────────────────
Falhas: nenhuma | Desvios: nenhum
════════════════════════════════════════
```

Atualizar STATE.md: loop position BUILD ✓ para cada refine.

Cleanup:
```
Clean up the team
```

Oferecer TEST:
```
Continue to TEST?
[1] Yes, run /orbti:test | [2] Pause here
```
</step>

<step name="handle_failures">
If a task verification fails:

1. **Stop immediately** - do not proceed to next task
2. **Report clearly:**
   - Which task failed
   - What verification check failed
   - What was expected vs actual
3. **Offer options:**
   - Retry: attempt the task again
   - Skip: mark as failed, continue (creates deviation)
   - Stop: halt execution, prepare for debugging
4. **Record resolution to STATE.md:**
   - Add to `### Decisions` section: `| [date]: Task [N] [retry/skip/stop] - [reason] | Project [N] | [impact] |`
</step>

<step name="track_progress">
Throughout execution:

1. Maintain mental log of:
   - Tasks completed (with results)
   - Tasks failed (with reasons)
   - Checkpoints resolved (with decisions/approvals)
   - Deviations from refine
2. This information feeds into INTEGRATE phase
</step>

<step name="finalize">
After all tasks attempted:

1. Summarize execution:
   - Tasks completed: N of M
   - Failures: list any
   - Deviations: list any

2. Update STATE.md e LOOP.md:
   - STATE.md: loop position REFINE ✓ → BUILD ✓ → INTEGRATE ○
   - Se LOOP.md existe: atualizar status da loop executada para "built — aguardando INTEGRATE"
   - Last activity: timestamp e loop executada

3. **Gate de verificação — obrigatório antes de declarar BUILD DONE:**

   **LEI: Não afirmar BUILD DONE sem evidência real de execução.**

   Para cada task concluída: mostrar o output dos comandos `<verify>`.
   Para testes: mostrar output completo (pass count, fail count, names).
   Para comandos: mostrar stdout/exit code.

   Frases proibidas sem evidência:
   - "deveria funcionar agora"
   - "provavelmente passa"
   - "parece correto"
   - "o agente reportou sucesso" → verificar independentemente

   Só após mostrar evidência real → reportar:

   **Se `type: front`:** extrair as rotas e localização no menu do REFINE-FRONT.md e incluir no output:
   ```
   ════════════════════════════════════════
   BUILD COMPLETE
   ════════════════════════════════════════
   [execution summary com output real das verificações]
   ────────────────────────────────────────
   Páginas disponíveis:
     http://localhost:3010/carteira/eficiencia/remessas
     http://localhost:3010/carteira/eficiencia/automacao

   Menu: Carteira → Eficiência de Débito → Remessas
   ────────────────────────────────────────
   Branch: fix/{slug} — visível no VS Code Source Control
   ```

   - `base_url` vem de `.orbti/config.md` (campo `e2e.base_url`). Se não configurado, usar `http://localhost:3000`.
   - **Localização no menu:** extrair do que foi implementado na task de sidebar — grupo pai → subitem. Formato: `{Seção} → {Grupo} → {Item}`. Ex: `Carteira → Eficiência de Débito → Remessas`.

   **Se `type: execute` (backend) ou sem páginas:** omitir URLs e menu.

4. **Informar worktree quando `isolation: "worktree"` foi usado:**

   Se o REFINE.md tinha `worktree.enabled: true` e o agente retornou `worktree_path` + `branch`:

   ```
   Mudanças em branch isolada:
     Branch: [branch]
     Path:   [worktree_path]
   Para revisar: cd [worktree_path] && git diff main
   ```

   O INTEGRATE cuidará do merge. Nesta fase, apenas informar.

   ```
   ────────────────────────────────────────
   Continue to TEST?

   [1] Yes, run /orbti:test | [2] Pause here
   ════════════════════════════════════════
   ```
   **Accept quick inputs:** "1", "yes", "test", "go" → run `/orbti:test [refine-path]`

   `/orbti:test` escreve os testes, executa, e roteia para INTEGRATE.
</step>

</process>

<output>
- Modified files as specified in REFINE.md
- Execution log (mental, for INTEGRATE)
- STATE.md updated with BUILD complete
</output>

<error_handling>
**Refine not found:**
- Check STATE.md for correct path
- Ask user to confirm refine location

**Boundary violation attempted:**
- Stop immediately
- Report which boundary would be violated
- Do not modify protected files

**Verification command fails:**
- Report the failure
- Offer retry/skip/stop options
- Do not mark task as complete

**Checkpoint timeout:**
- Remind user checkpoint is waiting
- Do not proceed without response
- Offer to save state and continue later
</error_handling>

<anti_patterns>
**Assuming approval:**
Do NOT start BUILD without explicit user approval of the refine.

**Writing tests during BUILD:**
BUILD does NOT write tests. Tests are written and executed by `/orbti:test` AFTER the human checkpoint. Keep BUILD focused on implementation only.

**Ignorar check_parallel_mode:**
Não pular o check de paralelo. Com `parallel_build: true` e múltiplos refines na loop, o modo correto é Agent Teams — ignorar silencia o paralelismo.

**Spawnar teammate para task bloqueada:**
Tasks com `depends_on` não resolvidos NÃO devem ser spawnadas. Aguardar o desbloqueio automático via task list antes de spawnar o teammate correspondente.

**Editar arquivos sobrepostos em paralelo:**
Dois teammates editando o mesmo arquivo causam sobrescrita. Garantir que os REFINEs paralelos toquem conjuntos de arquivos disjuntos (verificar `files_modified` no frontmatter antes de spawnar).

**Skipping verification:**
Every task MUST have its verify step run. No "it looks right" assumptions.

**Ignoring boundaries:**
If a task would modify a protected file, STOP. Do not rationalize violations.

**Proceeding past checkpoints:**
Checkpoints are blocking. Do not continue without user response.

**Creating a new REFINE after BUILD without INTEGRATE:**
After BUILD completes, the ONLY valid next steps are TEST or INTEGRATE.
Do NOT create a new REFINE.md while INTEGRATE ○ is open.
If the user asks to create another refine after BUILD, respond:
"Loop not closed — run /orbti:integrate [refine-path] first."
No exceptions.
</anti_patterns>

<workflow-dev>
# Decisões de Design — BUILD workflow

## Modelo de Loop (2026-04-08)

**Decisão:** Uma loop = um bloco de construção. Uma invocação de `/orbti:build` executa APENAS a próxima loop pendente.

**Rationale:** Loops são blocos autônomos e revisáveis. Avançar automaticamente para a próxima loop sem INTEGRATE+aprovação cria risco de divergência acumulada entre o que foi planejado e o que foi construído. O usuário decide conscientemente quando avançar.

**Comportamento:**
- Argumento arquivo → executa aquele refine EXCLUSIVAMENTE (sem orquestração)
- Argumento diretório → identifica a próxima loop pendente via LOOP.md, executa só ela

---

## depends_on = Comunicação, não Sequenciamento (2026-04-08)

**Decisão:** `depends_on` NÃO bloqueia o spawn do agente. Todos os REFINEs da loop são spawnados simultaneamente. `depends_on` define que há comunicação via SendMessage entre os agents.

**Rationale:** Front e back podem construir em paralelo — o front constrói com mocks, enquanto o back constrói a API. O `depends_on` indica o ponto onde o front precisa de informação do back (ex: "endpoint pronto em /v2/foo com esses types"). Bloquear o spawn seria desperdiçar paralelismo disponível.

**Protocolo de comunicação:**
- Builder dependente envia `PRECISO: <o que precisa>` quando chega ao ponto de integração
- Builder da dependência responde `PRONTO: <o que está disponível>` quando termina
- Se não houver resposta → continuar com mocks e reportar ao lead

---

## worktree = isolation: "worktree" nativo (2026-04-08)

**Decisão:** Usar o parâmetro `isolation: "worktree"` nativo do Agent tool do Claude Code. NÃO usar comandos bash para criar git worktrees manualmente.

**Rationale:** O Claude Code gerencia o worktree nativamente — cria o branch físico isolado, executa o agente nele, e retorna `worktree_path` + `branch` no resultado. Implementação manual via bash era frágil (symlinks, cleanup manual, race conditions entre agentes paralelos).

**Consequência:** O REFINE define `worktree: enabled: true` no frontmatter → o BUILD passa `isolation: "worktree"` no Agent tool. Sem configuração adicional necessária.

---

## autonomous = Confirmação por Task (2026-04-08)

**Decisão:** `autonomous: false` significa pausar após CADA task type="auto" para revisão e confirmação. `autonomous: true` executa tudo direto.

**Rationale:** `autonomous` controla o nível de supervisão humana durante o BUILD. Com `false`, o usuário revisa cada passo incrementalmente — útil para refines de alto risco ou que tocam código crítico. Com `true`, o agente corre autônomo e o usuário revisa apenas no BUILD COMPLETE.

**Consequência:** Refines com `autonomous: false` NÃO podem rodar em background (requerem humano a cada task).

---

## LOOP.md como Fonte de Verdade (2026-04-08)

**Decisão:** Quando um projeto tem 2+ loops, o LOOP.md é a fonte de verdade sobre a estrutura de construção. O BUILD lê LOOP.md antes de qualquer scan de arquivos.

**Rationale:** Evita reconstruir o grafo de loops a cada invocação por scanning de frontmatter. O LOOP.md documenta intenção (o que cada loop entrega) além de status técnico.

**Atualização:** BUILD atualiza LOOP.md ao completar uma loop (status → "built — aguardando INTEGRATE").
</workflow-dev>
