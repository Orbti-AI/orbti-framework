<purpose>
Create an executable REFINE.md for the current or specified project. The refine defines objective, acceptance criteria, tasks, boundaries, and verification - everything needed for BUILD phase execution.
</purpose>

<when_to_use>
- Starting a new project (ROADMAP shows next project ready)
- Previous refine completed (loop closed with INTEGRATE)
- First refine in a project (after init-project)
- Resuming work that needs a new refine
</when_to_use>

<loop_context>
Expected phase: REFINE
Prior phase:  INTEGRATE (previous refine complete) or none (first refine)
Next project: BUILD (after refine approval)
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/ROADMAP.md
@.orbti/PROJECT.md
@.orbti/projects/{prior-project}/{refine}-INTEGRATE.md (if exists and relevant)
</required_reading>

<references>
@./.claude/orbti-framework/references/refine-format.md
@./.claude/orbti-framework/references/checkpoints.md (if refine will have checkpoints)
@./.claude/orbti-framework/templates/REFINE.md
</references>

<process>

<step name="parse_specialist_flags" priority="first">
**Primeiro: detectar flags de specialist nos argumentos.**

## Flags disponíveis

| Flag | Significado |
|------|------------|
| `-F` | Incluir apenas Front |
| `-B` | Incluir apenas Back |
| `-T` | Forçar criação de refine Test (mesmo se config desabilitada) |
| `-A` | Incluir Agent |
| `--T` | Excluir Test (mesmo se config habilitada) |
| `--A` | Excluir Agent |
| (sem flags) | Todos os specialists detectados automaticamente |

**Regra do Test (T):**
Criar refine T SOMENTE se:
1. `test: enabled: true` em `.orbti/config.md`, **OU**
2. Flag `-T` passada explicitamente

Se nenhuma condição → não criar refine T (silencioso, sem aviso).

```bash
grep -A2 "^test:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Guardar flags detectadas para usar em `detect_layers`.
</step>

<step name="progressive_research">
**Aprofundar o research progressivo antes de criar os refines.**

---

## Detectar modo de entrada (determina profundidade do research)

```bash
ls .orbti/projects/{slug}/COCREATE.md 2>/dev/null && echo "HAS_COCREATE"
ls .orbti/projects/{slug}/OBSERVE.md  2>/dev/null && echo "HAS_OBSERVE"
```

### MODO A — COM COCREATE.md (research focado)

**1. Ler o Handoff do COCREATE — PRIMEIRO e ÚNICO ponto de entrada.**

```bash
cat .orbti/projects/{slug}/COCREATE.md 2>/dev/null | grep -A120 "## Handoff para Refine"
```

O handoff já resolve: o que ler, que HMW técnico abrir, que riscos endereçar, que protótipos referenciar.
**Não re-ler o COCREATE.md inteiro** — o handoff foi escrito exatamente para evitar isso.

Extrair do handoff:
- `Leia antes de criar tasks` → lista de arquivos a ler agora
- `HMW técnico` → guardar para step `technical_hmw`
- `Riscos` → guardar para usar nas tasks
- `Protótipos` → guardar para tasks de front

**2. Verificar RESEARCH.md acumulado:**
```bash
cat .orbti/projects/{slug}/RESEARCH.md 2>/dev/null
```
Não re-descobrir o que já está documentado.

**3. Pesquisar apenas o novo** (guiado pelos ponteiros do handoff):
- Ler os arquivos listados em `Leia antes de criar tasks`
- Confirmar estado atual de schemas/migrations referenciados
- Verificar dependências entre refines do loop

---

### MODO B — SEM COCREATE, COM OBSERVE.md (research médio)

**1. Ler `## Para a próxima fase` do OBSERVE.md — ponto de entrada do research:**

```bash
cat .orbti/projects/{slug}/OBSERVE.md 2>/dev/null | grep -A30 "## Para a próxima fase"
```

Esta seção contém os ponteiros deixados pelo OBSERVE: paths/módulos/tabelas específicos a investigar.
**Começar pelos ponteiros** — não explorar o codebase aleatoriamente.

**2. Verificar RESEARCH.md acumulado:**
```bash
cat .orbti/projects/{slug}/RESEARCH.md 2>/dev/null
```

**3. Research técnico completo** (guiado pelos ponteiros do OBSERVE):
- Arquivos, funções, hooks, componentes existentes relevantes
- Schemas, migrations, endpoints existentes
- Dependências entre módulos
- Padrões de implementação existentes no projeto

---

### MODO C — SEM OBSERVE E SEM COCREATE (refine standalone — research completo)

**Quando o usuário usa /orbti:refine diretamente sem fases anteriores.**

Sem handoffs para ler, o refine faz uma passagem completa de research — negócio + estratégico + técnico em um único passo. Não há o que reutilizar.

**Pesquisar em sequência:**
1. **Negócio**: domínios afetados, menus, fluxos existentes
2. **Estratégico**: dependências de construção, o que precisa existir antes de quê
3. **Técnico**: arquivos específicos, hooks, schemas, endpoints

**Não criar RESEARCH.md pré-existente** — o refine está criando a primeira camada.

---

## Adicionar ao RESEARCH.md (todos os modos)

```bash
# Criar se não existe
cat .orbti/projects/{slug}/RESEARCH.md 2>/dev/null || echo "## RESEARCH — {slug}\n\n> Research progressivo. Cada fase adiciona uma camada mais profunda.\n> OBSERVE → negócio | COCREATE → estratégico/UX | REFINE → técnico"
```

Adicionar seção:

```markdown
## Refine Loop N — {descrição} ({YYYY-MM-DD})

### Front (F)
- Arquivo: [path] — [o que contém de relevante]
- Componente: [nome] — [como reutilizar/estender]
- Hook: [nome] — [assinatura, o que faz]

### Back (B)
- Controller: [path] — [endpoints existentes]
- Service: [path] — [lógica existente]
- Migration: [última] — [estado atual do schema]

### Perguntas respondidas
- [Pergunta do observe/cocreate]: [resposta técnica]

### Perguntas abertas
- [O que ainda não ficou claro]

### Para a próxima fase
Ponteiros para o próximo loop (refine do Loop N+1):
- `[path]` — [o que confirmar / o que a resposta impacta no próximo loop]
```

**Regra de "Para a próxima fase":** preencher se há loop subsequente planejado. Deixar os ponteiros que o próximo refine vai precisar — evita que o Loop 2 repita buscas que o Loop 1 já fez.

**Silencioso:** executar sem anunciar ao usuário. Exibir apenas o resultado final.
</step>

<step name="technical_hmw">
**HMW técnico — posição condicional ao COCREATE.md.**

```bash
cat .orbti/projects/{slug}/COCREATE.md 2>/dev/null | head -5
```

---

### COM COCREATE.md — HMW upfront + HMW residual

**Antes do research:** ler `COCREATE.md` → extrair decisões técnicas abertas (seção "Handoff para Refine" e qualquer item marcado como "a resolver") → abrir HMW técnico imediatamente, ANTES de pesquisar.

O gestor já sabe o que não sabe. Não faz sentido esperar.

**Durante o research:** anotar novos unknowns que surgirem — não interromper com perguntas a cada descoberta.

**Após o research:** se o research revelou novos unknowns além dos iniciais → abrir rodada de HMW residual antes de criar as tasks.

```
Fluxo: HMW iniciais (COCREATE) → research → HMW residuais (se houver) → tasks
```

---

### SEM COCREATE.md — HMW após research

Sem base estratégica, HMW upfront seria genérico. Fazer research primeiro, depois abrir HMW com base no que foi descoberto.

```
Fluxo: research → HMW técnico → tasks
```

---

### Escopo do HMW técnico (ambos os casos)

Perguntas que o COCREATE não faz — exigem conhecer o codebase:
- Trade-offs de implementação: "reutilizar X ou criar Y?"
- Idempotência, race conditions, edge cases específicos
- Acoplamento: "injetar direto ou via queue?"
- Schema: coluna nova vs campo JSONB existente
- Camada: domain service vs use case vs infra?

**Pular silenciosamente** se não há decisão técnica não óbvia — não forçar HMW onde tudo é evidente.

**Formato:**
```
HMW técnico — {slug} Loop {N}

1. Como implementar {X} — [opção A] vs [opção B]?
   Contexto: {o que o research/COCREATE revelou}

2. Como garantir {Y} considerando {constraint encontrado}?

────────────────────────────────────────
Responda as relevantes — as respostas entram nas tasks do refine.
```
</step>

<step name="detect_layers" priority="second">
**Detectar specialists do loop e aplicar flags.**

## Como analisar

Ler o pedido do usuário (argumentos + OBSERVE.md + COCREATE.md se existir). Para cada entregável mencionado, classificar:

**É frontend se o entregável é:**
- Uma tela, página, dashboard, painel, modal, formulário, componente visual
- Um protótipo passado como referência: link Paper/Figma/Stitch, arquivo .pen, HTML
- Um componente React ou qualquer coisa que roda no browser e o usuário vê

**Detecção de protótipo (junto com a classificação de layer):**
Se o input contém qualquer sinal de protótipo (URL de design tool, arquivo .pen/.html, MCP de design ativo), extrair e guardar `prototype` + `prototype_tool` para uso no frontmatter do LL_SS-F-REFINE.md.

**É backend se o entregável é:**
- Um endpoint, rota de API, controller, service
- Uma operação no banco (migration, query, schema)
- Lógica de negócio, regra de cálculo, processamento que roda no servidor
- Worker, job, evento, integração com serviço externo

**Quando há dúvida:** perguntar — melhor do que colocar na layer errada.

## Exemplos de classificação

```
"criar tela de listagem de remessas"           → FRONT
"criar endpoint GET /remessas"                 → BACK
"implementar filtro de banco por data"         → BACK
"mostrar filtro de data na tela"               → FRONT
"refatorar o componente de tabela"             → FRONT
"criar migration para adicionar coluna"        → BACK
"tela + api para cadastro de parceiros"        → FRONT + BACK
"protótipo HTML do dashboard"                  → FRONT
"link do Figma para implementar"               → FRONT
```

## Decisão (aplicando flags de specialist)

```
Só frontend  → 01_01-F-REFINE.md
Só backend   → 01_01-B-REFINE.md
Ambos        → 01_01-B-REFINE.md + 01_02-F-REFINE.md (paralelo)
Com test     → 01_03-T-REFINE.md (apenas se config test habilitada ou flag -T)
Com agent    → LL_SS-A-REFINE.md
```

Exibir antes de continuar:
```
Specialists detectados: [F | B | T | A] (após aplicar flags)

  01_01-B-REFINE.md  — {tasks de back}
  01_02-F-REFINE.md  — {tasks de front}
  01_03-T-REFINE.md  — {cenários de teste} [se test habilitado]
  Loop 1 — paralelo (F + B + A simultâneos, T após ou junto)
```

**Se só backend:** ir para `validate_preconditions` normalmente.
**Se há frontend:** ir para `create_front_refine` (depois voltar para `validate_preconditions` se também há backend).
</step>

<step name="create_front_refine">
**Criar NN-loopN-F-REFINE.md para o trabalho de frontend.**

Nome: `{NN}-NN-loopN-F-REFINE.md`
Local: `.orbti/projects/{slug}/`

---

## Sub-step: map_ui_components

**Executar ANTES de escrever qualquer task. Sempre.**

O objetivo é garantir que o NN-loopN-F-REFINE.md especifique componentes concretos — não "criar tabela" mas "usar `<Table>` de `@/components/ui/table`". Reuso de componentes existentes é a regra, construção de novos é a exceção.

### 1. Verificar se COMPONENTS.md existe

Detectar o submodule frontend do projeto (geralmente `crm-menos-juros` para telas do CRM, `agent` para interfaces do agent, etc.) e checar:

```bash
# Checar no submodule detectado primeiro, depois na raiz do codebase
ls .orbti/codebase/{submodule}/COMPONENTS.md 2>/dev/null \
  || ls .orbti/codebase/COMPONENTS.md 2>/dev/null
```

**Se existe:** usar como fonte primária. Ler o arquivo completo. Não re-escanear o que já está mapeado.
**Se não existe:** escanear diretamente (passo 2).

> COMPONENTS.md é gerado pelo `/orbti:map-codebase` — se não existe, considerar rodar o mapeamento primeiro para acelerar futuros refines.

### 2. Escanear componentes disponíveis (quando COMPONENTS.md ausente)

```bash
# shadcn/ui instalados
ls apps/crm-menos-juros/client/src/components/ui/ 2>/dev/null

# Componentes customizados reutilizáveis
ls apps/crm-menos-juros/client/src/components/ 2>/dev/null
```

Ler os arquivos dos componentes relevantes para o scope da tela (não todos — apenas os candidatos).

### 3. Identificar elementos UI da tela

A partir do input (protótipo, link de design, HTML, descrição), listar cada elemento UI distinto:
- Tabelas, listas, cards, formulários, modais, sheets
- Badges, botões, inputs, selects, tabs, alerts
- Navegação, headers, layouts

### 4. Mapear cada elemento a um componente

Para cada elemento, classificar em uma de três categorias:

| Categoria | Quando | Ação no REFINE |
|-----------|--------|----------------|
| **Reutilizar** | Componente existente resolve completamente | Especificar import + variante exata na task |
| **Estender** | Componente existe mas precisa de wrapper ou variante nova | Especificar base + o que adicionar |
| **Criar** | Não existe equivalente — ou o protótipo define algo fundamentalmente diferente | Dar direcionamento seguindo o protótipo: estrutura, props esperadas, tokens a usar |

### 5. Produzir Component Map

Incluir no NN-loopN-F-REFINE.md como seção `## Component Map`:

```markdown
## Component Map

| Elemento | Status | Componente | Import |
|----------|--------|------------|--------|
| Tabela de bancos | reutilizar | `<Table>` + TanStack | `@/components/ui/table` |
| Badge status | reutilizar | `<Badge variant="destructive\|warning\|success">` | `@/components/ui/badge` |
| Sheet de detalhes | reutilizar | `<Sheet>` | `@/components/ui/sheet` |
| Régua de efetividade | criar | Novo: `<EfetividadeRuler>` | — |
| Alerta de ação | estender | `<Alert>` + border-left 4px override | `@/components/ui/alert` |
```

**Regra de criação:** quando criar é necessário, incluir na task:
- Nome do componente (PascalCase, sufixo pela função: `EfetividadeRuler`, `KpiCard`)
- Props mínimas necessárias (tipadas)
- Tokens do design system a usar (de `.orbti-design/system.md`)
- Estrutura HTML/JSX do sketch do protótipo como referência

**Sem protótipo:** maximizar reutilização. Se não há referência visual, criar apenas quando nenhum componente existente serve — e nesse caso, seguir o padrão mais próximo já existente no codebase.

---

## Frontmatter

```yaml
---
project: {slug}
refine: "NN"
specialist: F          # F=Front | B=Back | T=Test | A=Agent
loop: N                # Número do loop (1, 2, 3...)
type: execute          # execute | tdd | research
depends_on: []
integrates_with: ""    # ID do refine B do mesmo loop (ex: "02") — omitir se não há
autonomous: false
prototype: ""          # URL ou caminho do protótipo — omitir se não há
prototype_tool: ""     # paper | pencil | figma | stitch | html | outro
---
```

## Naming Convention

```
LL_SS-S-REFINE.md
```

| Parte | Significado | Exemplo |
|-------|------------|---------|
| `LL` | Número do loop | `01`, `02`, `03` |
| `SS` | Sequencial dentro do loop (evita colisão de nomes) | `01`, `02`, `03` |
| `S` | Specialist | `F`, `B`, `T`, `A` |

Exemplos: `01_01-B-REFINE.md`, `01_02-F-REFINE.md`, `02_01-B-REFINE.md`, `02_02-F-REFINE.md`

Loop com B + F paralelos: `02_01-B-REFINE.md` + `02_02-F-REFINE.md`

### Preenchimento de `integrates_with`

Quando um REFINE B (`specialist: B`) é criado **no mesmo loop** que este REFINE F:
- Preencher `integrates_with` com o nome do arquivo REFINE B (ex: `"02_01-B-REFINE.md"`)
- Omitir se o frontend não depende de backend novo nesta loop

O BUILD usa `integrates_with` para saber qual backend aguardar antes da integração real.

### Detecção de protótipo

Ao processar o input do usuário, detectar automaticamente:

| Sinal | Tool | Exemplo |
|-------|------|---------|
| URL `app.paper.design/...` | paper | `https://app.paper.design/file/...` |
| Arquivo `.pen` | pencil | `designs/tela.pen` |
| URL `figma.com/...` ou `figma.com/proto/...` | figma | `https://www.figma.com/file/...` |
| URL `stitch.withgoogle.com/...` | stitch | `https://stitch.withgoogle.com/...` |
| Arquivo `.html` ou trecho HTML inline | html | `tela.html` ou `<div>...</div>` |
| MCP Paper ativo (`mcp__paper__*`) | paper | artboard aberto no Paper Desktop |
| MCP Pencil ativo (`mcp__pencil__*`) | pencil | arquivo .pen aberto no Pencil |
| MCP Figma ativo | figma | arquivo aberto via plugin Figma |

Se protótipo detectado:
- Salvar `prototype` com o valor exato passado (URL, caminho, ou descrição do artboard)
- Salvar `prototype_tool` com o nome da ferramenta
- O BUILD usará esses campos para saber onde buscar a referência visual

Se nenhum protótipo detectado: omitir ambos os campos do frontmatter.

## Conteúdo

Seguir o mesmo template de REFINE.md, mas escopo restrito a frontend:

- `<objective>`: O que construir na UI — telas, componentes, fluxos
- `<input>`: O que foi passado como referência (link Figma, arquivo .pen/.paper, HTML, descrição)
  - Se link/arquivo de design: referenciar diretamente — o BUILD usará como âncora
  - Se descrição: descrever o que a tela deve mostrar e como
- `<component_map>`: tabela de mapeamento gerada no sub-step map_ui_components (obrigatória)
- `<context>`: @-referência ao design system (`.orbti-design/system.md`) se existir
- `<acceptance_criteria>`: critérios visuais e funcionais do frontend (sem backend real — usar dados mockados se backend ainda não existe)
- `<tasks>`: tasks de frontend — cada task referencia componentes concretos do component map
  - "criar tabela" → "criar página usando `<Table>` de `@/components/ui/table` com TanStack Table"
  - "criar badge" → "usar `<Badge variant='destructive'>` para status Crítico"
  - "criar régua" → "criar componente `<EfetividadeRuler>` com props value/max, usar `--success/--warning/--destructive`"
- `<boundaries>`:
  - NÃO fazer: lógica de backend, migrations, endpoints novos
  - Se backend ainda não existe: usar dados mockados nas tasks de frontend

## Após criar

Se o pedido também tem backend → continuar para `validate_preconditions` (cria o REFINE.md backend).
Se só frontend → ir direto para `validate_plan`.
</step>

<step name="validate_preconditions" priority="first">
**HARD BLOCK — run before anything else. Do not skip under any circumstances.**

## Modelo mental: REFINEs são fases

REFINEs são fases independentes de um projeto. Cada fase tem seu próprio loop BUILD → INTEGRATE.
Fases podem ser **paralelas** (loop: 1, depends_on: []) ou **sequenciais** (loop: 2, depends_on: ['01']).
O BUILD detecta todas as fases pendentes e executa em paralelo quando `parallel_build: true`.

## Regras de validação

1. Read STATE.md
2. Check `parallel_build` config:

   ```bash
   grep "parallel_build:" .orbti/config.md 2>/dev/null -A1 | grep "enabled: true"
   ```

3. **Se `parallel_build: false` (modo sequencial):**

   **If loop shows BUILD ✓ and INTEGRATE ○ (open loop):**
   ```
   ⛔ BLOCKED: Loop not closed
   ════════════════════════════════════════
   BUILD is complete but INTEGRATE has not run.
   You cannot create a new REFINE until the current loop is closed.

   Required next step:
   ▶ /orbti:integrate [current-refine-path]
   ════════════════════════════════════════
   ```
   **Stop. Do not create REFINE.md.**

   **If loop shows REFINE ✓ and BUILD ○ (refine not yet built):**
   ```
   ⛔ BLOCKED: Existing refine pending
   ════════════════════════════════════════
   A REFINE already exists and has not been built yet.
   Build it before creating a new one (sequential mode).

   Required next step:
   ▶ /orbti:build [existing-refine-path]
   ════════════════════════════════════════
   ```
   **Stop. Do not create a second REFINE.md.**

4. **Se `parallel_build: true` (modo paralelo — padrão):**

   **If loop shows BUILD ✓ and INTEGRATE ○ (open loop):**
   ```
   ⛔ BLOCKED: Loop not closed
   ════════════════════════════════════════
   BUILD is complete but INTEGRATE has not run.
   Close the loop before creating new REFINEs.

   Required next step:
   ▶ /orbti:integrate [current-refine-path]
   ════════════════════════════════════════
   ```
   **Stop. INTEGRATE is still required before new phases.**

   **If loop shows REFINE ✓ and BUILD ○ (existing phases pending):**
   → Permitido. Novas fases podem ser criadas livremente.
   → O BUILD detectará todas as fases pendentes e executará em paralelo.
   → Informar ao usuário:
   ```
   ℹ Parallel mode: existing phases pending BUILD.
   Creating new phase — BUILD will execute all phases in parallel.
   ```
   → Continuar normalmente para `identify_phase`.

5. Proceed normally if: no open loops, or parallel_build: true with only pending REFINEs.
</step>

<step name="identify_phase">
1. Read ROADMAP.md to determine:
   - Which project is next (first incomplete project)
   - Project scope and goals
   - Dependencies on prior projects
2. If multiple projects available, ask user which to refine
3. Confirm project selection before proceeding
</step>

<step name="detect_worktree_flag">
**Parse flags `-w` e `-b` from $ARGUMENTS before anything else.**

Padrões aceitos:
```
-w [branch] [description]
-w [branch] -b [base] [description]
```

Exemplos:
```
-w fix/bug-parcelas "Corrigir cálculo de parcelas vencidas"
-w fix/bug-parcelas -b main "Corrigir cálculo de parcelas vencidas"
-w feature/remessa-cnab -b feature/carteira "Implementar geração de CNAB"
```

Se `-w` presente:
1. Extrair `worktree_branch` (token após `-w`)
2. Extrair `base_branch` (token após `-b`, **default: `main`** se não especificado)
3. Extrair `description` (texto restante)
4. Armazenar os três para uso em `create_plan` e `update_state`
5. **Validate `autonomous` requirement:** worktree loop requer `autonomous: true`
   - Será verificado em `analyze_scope` — se checkpoints necessários, avisar:
     ```
     ⚠ Modo worktree requer autonomous: true.
     O refine que você está planejando precisa de checkpoints.
     [1] Remover checkpoints e manter -w
     [2] Continuar sem -w (fluxo normal com checkpoints)
     ```

Se `-w` ausente: seguir normalmente, `worktree_branch` = nil.
</step>

<step name="analyze_scope">
1. Review project goals from ROADMAP.md
2. Estimate number of tasks needed:
   - Target: 2-3 tasks per refine
   - If >3 tasks, consider splitting into multiple refines
3. Identify files that will be modified
4. Determine if checkpoints are needed:
   - Visual verification required? → checkpoint:human-verify
   - Architecture decision needed? → checkpoint:decision
   - Unavoidable manual action? → checkpoint:human-action (rare)
5. Set autonomous flag: true if no checkpoints, false otherwise
6. If `worktree_branch` set and checkpoints needed → enforce conflict resolution (see `detect_worktree_flag`)
</step>

<step name="detect_phase_position">
**Determinar loop e depends_on antes de criar o REFINE. Sempre executar.**

### 1. Escanear REFINEs existentes no projeto

```bash
ls .orbti/projects/{project-name}/*-REFINE.md 2>/dev/null
```

Extrair de cada REFINE existente: `refine`, `loop`, `depends_on`, `files_modified`.

### 2. Classificar o novo REFINE

**Se não há REFINEs existentes:**
→ `loop: 1`, `depends_on: []` (primeira fase)

**Se há REFINEs existentes**, analisar:

**Candidato a PARALELO (loop igual ao maior existente, depends_on: []) se:**
- Nenhum arquivo em `files_modified` se sobrepõe com outros REFINEs pendentes de BUILD
- O trabalho não depende de output de outro REFINE (tipos, exports, schema, etc.)
- Pode começar imediatamente sem esperar outro agente

**Candidato a SEQUENCIAL (loop + 1, depends_on: [N]) se:**
- Compartilha arquivos com REFINE existente → conflito de edição
- Usa tipos, exports ou schema criados por outro REFINE
- Logicamente depende de resultado de outro REFINE para funcionar

### 3. Apresentar posicionamento antes de criar

```
════════════════════════════════════════
POSICIONAMENTO DA FASE
════════════════════════════════════════

Fases existentes no projeto:
  [01] feature X — loop 1, independente  (pendente BUILD)
  [02] fix Y     — loop 1, independente  (pendente BUILD)

Nova fase: [descrição do trabalho]

Posicionamento detectado: PARALELA
  loop: 1
  depends_on: []
  Motivo: arquivos sem sobreposição, sem dependência de output

[1] Confirmar paralela  [2] Tornar sequencial após [N]  [3] Ajustar
════════════════════════════════════════
```

Se o posicionamento for óbvio e sem conflitos → perguntar apenas se necessário.
Se há conflito de arquivos ou dependência clara → mostrar sempre e aguardar confirmação.

### 4. Armazenar

Guardar `loop` e `depends_on` para uso em `create_plan`.

### 5. Criar ou atualizar LOOP.md

**Se há 2+ loops no projeto** (loop atual > 1, OU há outros REFINEs em loop diferente):

Verificar se LOOP.md existe:
```bash
ls .orbti/projects/{projeto}/LOOP.md 2>/dev/null
```

**Se não existe:** criar usando template `.claude/orbti-framework/templates/LOOP.md`
- Listar todas as loops e REFINEs conhecidos (existentes + o novo que está sendo criado)
- Incluir: número de tasks por REFINE (contar `<task>` no REFINE.md), tipo, depends_on, worktree, status atual

**Se já existe:** atualizar — adicionar a nova loop/refine à tabela correspondente

O LOOP.md é a fonte de verdade sobre a estrutura de construção do projeto. Mantê-lo atualizado a cada novo REFINE criado.
</step>

<step name="load_context">
1. Read PROJECT.md for:
   - Core requirements and constraints
   - Value proposition (what matters)
2. If prior project exists, read its INTEGRATE.md for:
   - What was built
   - Decisions made
   - Any deferred issues
3. Read source files relevant to this phase's work
5. Do NOT reflexively chain all prior summaries - only load what's genuinely needed
</step>

<step name="check_specialized_flows">
**Check for SPECIAL-FLOWS.md, invoke required skills, and populate skills section.**

1. Check if `.orbti/SPECIAL-FLOWS.md` exists
2. If exists:
   - Read SPECIAL-FLOWS.md
   - Extract skills marked as "required" for the work type being planned
   - Match against phase/refine work being done
   - Prepare <skills> section content for REFINE.md
3. If not exists:
   - Add comment: "No SPECIAL-FLOWS.md - skills section omitted"
   - Skip skills section in REFINE (or include minimal placeholder)
4. **If required skills found — invoke them NOW via Skill tool before creating the plan:**
   - For each required skill, call the Skill tool to load it into context
   - This ensures the plan is created with the skill's knowledge already active
   - Inform the user which skills were loaded:
   ```
   ════════════════════════════════════════
   Skills carregadas para este refine:
   ════════════════════════════════════════
   ✓ /skill-1 (work type: X) — carregada
   ✓ /skill-2 (work type: Y) — carregada
   ════════════════════════════════════════
   ```

**Note:** Skills são invocadas ANTES de criar o plano — o REFINE.md resultante
já reflete o conhecimento das skills (estrutura correta, padrões aplicados).
Required skills will also BLOCK build if not confirmed loaded.
</step>

<step name="check_parallel_split">
**Só executa se `parallel_build.enabled: true` no config.**

```bash
grep "parallel_build:" .orbti/config.md 2>/dev/null -A1 | grep "enabled: true"
```

Se não encontrar → pular, ir para `create_plan` (single REFINE normal).

**Se parallel habilitado:**

Analisar o escopo do projeto (já lido em `analyze_scope`) e identificar **unidades independentes de trabalho** — grupos de tasks que não compartilham arquivos e podem ser executados sem esperar o outro.

Critérios de split:
- Tasks tocam subsistemas distintos (ex: API vs frontend, migration vs endpoint, script vs dashboard)
- Nenhum arquivo em comum entre os grupos (verificar `files_modified` estimados)
- Uma unidade não depende de output da outra para iniciar

Se só há 1 unidade coesa (ex: tudo no mesmo serviço, arquivos entrelaçados) → pular, ir para `create_plan`.

**Se 2+ unidades independentes identificadas:**

Apresentar proposta ao usuário antes de criar os arquivos:

```
════════════════════════════════════════
PARALLEL SPLIT DETECTADO
════════════════════════════════════════

parallel_build está ativo. Identifiquei [N] unidades independentes:

Unidade A — [nome] ([subsistema])
  Files: [lista]
  Tasks: [resumo]

Unidade B — [nome] ([subsistema])
  Files: [lista]
  Tasks: [resumo]
  depends_on: [] | [Unidade A] ← se houver dependência

Cada unidade vira um REFINE.md separado.
No BUILD, um agente por REFINE (em paralelo quando sem depends_on).

[1] Confirmar split  [2] Criar único REFINE (sequencial)  [3] Ajustar divisão
════════════════════════════════════════
```

Aguardar resposta antes de criar os arquivos.

**Se confirmado (opção 1):**

- Criar um REFINE.md por unidade com numeração sequencial: `01-REFINE.md`, `02-REFINE.md`, etc.
- Todos na mesma `loop: 1` se independentes entre si
- Unidades com dependência recebem `loop: 2` e `depends_on: [01]`
- Cada REFINE.md contém apenas as tasks, ACs, boundaries e files da sua unidade
- Pular `create_plan` (já criado aqui)
- Ir para `validate_plan` e `update_state`

**Se opção 2:** Pular, ir para `create_plan` (single REFINE normal).

**Se opção 3:** Aguardar nova divisão do usuário, re-apresentar proposta.
</step>

<step name="create_plan">
1. Create project directory: `.orbti/projects/{project-name}/`
   **IMPORTANT:** `{project-name}` is the project name in kebab-case with NO numeric prefix.
   Even if the ROADMAP lists projects as "01. cohort-inadimplencia", use only "cohort-inadimplencia".
   WRONG: `.orbti/projects/01-cohort-inadimplencia/`
   RIGHT: `.orbti/projects/cohort-inadimplencia/`

2. **Naming convention:** nome: `01-REFINE.md` (padrão)

3. Generate REFINE.md following template structure:

   **Frontmatter:**
   - project: name
   - refine: 01 (or next number if multiple refines in phase)
   - type: execute (or tdd/research)
   - loop: 1 (adjust if dependencies exist)
   - depends_on: [] (or prior refine IDs if genuine dependency)
   - files_modified: [list all files]
   - autonomous: true/false
   - worktree_branch: [branch] (only if `-w` flag was used — omit otherwise)
   - worktree_base: main (default) ou branch especificada em `-b` — omit if no `-w`

   **Sections:**
   - <objective>: Goal, Purpose, Output
   - <context>: @-references to project files and source
   - <acceptance_criteria>: Given/When/Then for each criterion
   - <tasks>: Task definitions with files, action, verify, done
   - <boundaries>: DO NOT CHANGE, SCOPE LIMITS
   - <verification>: Overall completion checks
   - <success_criteria>: Measurable completion
   - <output>: INTEGRATE.md location

3. Ensure every task has:
   - Clear files list
   - Specific action (not vague)
   - Verification command/check
   - Done criteria linking to AC-N

**Regra absoluta: zero placeholders.**
Um REFINE.md com "TBD", "implementar depois", "adicionar validação", "TODO", "a definir"
é um REFINE INCOMPLETO — não apresentar ao usuário.
Cada task deve estar completamente especificada.
Se não souber como implementar algo → pesquisar antes de criar o REFINE.

**Para features complexas ou com múltiplas abordagens possíveis:**
Antes de definir as tasks, propor 2-3 abordagens com trade-offs.
Apresentar ao usuário e aguardar escolha.
Só após escolha definida → detalhar as tasks.
Isso evita retrabalho de planejar a implementação errada.
</step>

<step name="validate_plan">
**Self-review obrigatório antes de apresentar ao usuário:**

- [ ] **Cobertura de spec:** cada AC tem pelo menos uma task correspondente
- [ ] **Scan de placeholders:** nenhum "TBD", "TODO", "implement later", "adicionar validação"
- [ ] **Tasks específicas:** cada task tem files + action + verify + done preenchidos
- [ ] **Consistência de tipos:** nomes de funções/tipos usados em tasks diferentes coincidem
- [ ] **Boundaries completos:** arquivos que NÃO devem ser tocados estão explicitamente listados
- [ ] **Checkpoints necessários:** posicionados após trabalho automatizável, antes de decisões humanas

Se qualquer item falhar → corrigir antes de apresentar.
Não usar o usuário como revisor de placeholders ou tarefas incompletas.
</step>

<step name="update_state" priority="required">
**This step is REQUIRED. Do not skip.**

1. **Update STATE.md** with exact content:

   ```markdown
   ## Current Position

   Milestone: v0.1 [Milestone Name]
   Project: [N] of [total] ([Project Name]) — Planning
   Refine: [NN-PP] created, awaiting approval
   Status: REFINE created, ready for BUILD
   Last activity: [timestamp] — Created [refine-path]

   Progress:
   - Milestone: [░░░░░░░░░░] X%
   - Project [N]: [░░░░░░░░░░] 0%

   ## Loop Position

   Current loop state:
   ```
   REFINE ──▶ BUILD ──▶ INTEGRATE
     ✓        ○        ○     [Refine created, awaiting approval]
   ```

   ## Session Continuity

   Last session: [timestamp]
   Stopped at: Refine [NN-PP] created
   Next action: Review and approve refine, then run /orbti:build [refine-path]

   Resume file: [refine-path]
   ```

2. **Update ROADMAP.md** milestone status:
   - If first refine of milestone: Change "Not started" → "In progress"
   - Update project status: "Not started" → "Planning"

3. **Report with quick continuation prompt:**

   **Single REFINE (normal):**
   ```
   ════════════════════════════════════════
   REFINE CREATED
   ════════════════════════════════════════

   Refine: [refine-path]
   Project: [N] — [Project Name]

   [refine summary - key tasks, checkpoints]

   ---
   Continue to BUILD?

   [1] Approved, run BUILD | [2] Questions first | [3] Pause here
   ```

   **Múltiplos REFINEs (parallel split):**
   ```
   ════════════════════════════════════════
   REFINES CRIADOS — PARALLEL BUILD
   ════════════════════════════════════════

   Project: [N] — [Project Name]
   REFINEs: [N] arquivos na loop [W]

   [01-REFINE.md] [nome-unidade-a] — loop 1, independente
     Tasks: [N] | Files: [lista]

   [02-REFINE.md] [nome-unidade-b] — loop 1, independente
     Tasks: [N] | Files: [lista]

   [03-REFINE.md] [nome-unidade-c] — loop 2, depends_on: [01]
     Tasks: [N] | Files: [lista]

   No BUILD: agentes paralelos para loop 1,
   loop 2 aguarda loop 1 completar.

   ---
   Continue to BUILD?

   [1] Approved, run BUILD | [2] Questions first | [3] Pause here
   ```

4. **Accept quick inputs:**

   **Se `worktree_branch` está definido no REFINE.md:**
   - "1", "approved", "yes", "go" → spawn background agent com `isolation: "worktree"` rodando loop completo:
     ```
     Spawn background agent:
     isolation: worktree
     branch: [worktree_branch]

     PRIMEIRA AÇÃO — sincronizar com a base branch antes de qualquer coisa:
       git fetch origin
       git reset --hard origin/[worktree_base]
     Isso garante que o trabalho parte de [worktree_base] (default: main),
     independente da branch em que o usuário estava quando invocou o refine.

     Execute o loop completo para o refine em [refine-path]:
     1. BUILD: execute todas as tasks (workflow build.md → execute_tasks)
     2. TEST: execute os testes (workflow test.md)
     3. INTEGRATE: feche o loop (workflow integrate.md)

     Ao final:
     - Registre worktree_path e branch retornados pelo isolation
     - Reporte ao usuário com sumário do loop + opções de merge/discard
     ```
     Confirmar ao usuário:
     ```
     ════════════════════════════════════════
     LOOP COMPLETO RODANDO EM BACKGROUND
     ════════════════════════════════════════
     Refine: [refine-path]
     Worktree: [worktree_branch]

     Build → Test → Integrate rodando isolado.
     Sua branch atual não será afetada.
     Você será notificado quando o loop completar.
     ════════════════════════════════════════
     ```

   **Se `worktree_branch` não está definido (fluxo normal):**
   - "1", "approved", "yes", "go" → run `/orbti:build [project-dir]` (BUILD detecta todos os REFINEs)

   **Ambos os modos:**
   - "2", "questions" → wait for user questions, then re-offer the 3 options
   - "3", "pause" → run `/orbti:pause`
</step>

</process>

<output>
REFINE.md at `.orbti/projects/{project-name}/{refine}-REFINE.md`

Example: `.orbti/projects/workflows-layer/01-REFINE.md`

---

**Se `test_writer.enabled: true` no config — criar também o TEST-PLAN.md:**

Imediatamente após criar o REFINE.md, criar `{refine}-TEST-PLAN.md` na mesma pasta.

Para cada AC do REFINE.md, mapear:
- Cenários: happy path + edge cases relevantes
- Tipo: `unit` | `integration` | `e2e` | `manual`
- Checkpoint: `auto` (roda sem humano) | `human` (precisa aprovação)
- Dados: seletores, fixtures, usuário de teste, URLs, mocks necessários
- Runner: inferir de `.orbti/TESTS.md` (módulo afetado)

Template em: @./.claude/orbti-framework/workflows/test-auto.md (seção `create_test_plan`)
</output>

<error_handling>
**STATE.md missing:**
- Offer to create from ROADMAP.md inference
- Or ask user to run init-project first

**ROADMAP.md missing:**
- Cannot create refine without roadmap
- Ask user to create ROADMAP.md or run init-project

**Phase dependencies not met:**
- Warn user which prior phases must complete first
- Do not create refine until dependencies satisfied
</error_handling>

<workflow-dev>
# Decisões de Design — REFINE workflow

## Separação Front/Back sempre em arquivos distintos (2026-04-08)

**Decisão:** Front e back são SEMPRE REFINEs separados, na mesma loop, sem `depends_on` entre si por padrão.

**Rationale:** Front e back podem ser construídos em paralelo — o front constrói com mocks, o back constrói a API. Separar em arquivos distintos é o mecanismo pelo qual o BUILD sabe que pode spawnar dois agents simultaneamente. Um arquivo único forçaria execução sequencial.

**Exceção:** Se o front precisa de tipos gerados pelo back (ex: schema TypeScript exportado), adicionar `integrates_with: "{NN}-back"` ao REFINE-FRONT para que o BUILD saiba estabelecer comunicação via SendMessage entre os agents.

---

## integrates_with = Dependência de Integração (2026-04-08)

**Decisão:** Campo `integrates_with` no frontmatter do REFINE-FRONT declara qual REFINE de backend ele integra na fase 2.

**Rationale:** Sem este campo, o BUILD precisaria inferir a relação entre front e back por heurísticas (mesma loop, tipos em comum, etc.). O campo torna a intenção explícita e o BUILD pode usar para:
1. Estabelecer canal de comunicação SendMessage entre os agents
2. Saber quando o front deve aguardar sinal do back para fase 2 (integração real)
3. Documentar a dependência claramente no LOOP.md

**Regra:** Só preencher quando há REFINE de backend NA MESMA LOOP que este front precisa integrar. Omitir caso contrário.

---

## map_ui_components = Componentes Concretos (2026-04-08)

**Decisão:** Sub-step obrigatório antes de escrever qualquer task do REFINE-FRONT. Lê COMPONENTS.md do codebase map primeiro; escaneia diretamente como fallback.

**Rationale:** Tasks vagas ("criar tabela", "adicionar badge") levam a builders que criam componentes do zero quando existentes servem. O component map força o REFINE a especificar componentes concretos: import path, variante, categoria (reutilizar/estender/criar).

**Fonte primária:** `.orbti/codebase/{submodule}/COMPONENTS.md` — gerado por `/orbti:map-codebase`. Se não existe, escanear diretamente. Considerar rodar map-codebase primeiro para acelerar futuros refines.

---

## LOOP.md = Mapa de Construção (2026-04-08)

**Decisão:** Criar LOOP.md no projeto quando há 2+ loops. Atualizar a cada novo REFINE criado.

**Rationale:** O LOOP.md documenta a intenção de construção em blocos — não apenas status técnico. Permite ao `/orbti:progress` mostrar breakdown por loop com contagem de tasks, e ao `/orbti:build` identificar a próxima loop pendente sem reconstruir o grafo.

**Conteúdo:** Por loop: nome descritivo, lista de REFINEs, task count, depends_on, worktree, status atual. Mais barra de progresso visual no final.
</workflow-dev>
