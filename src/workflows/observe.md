<purpose>
Mapear o cenário atual e o cenário futuro antes de qualquer planejamento. O observe é o momento de entender **o que existe hoje** e **onde queremos chegar** — a diferença entre os dois é o que precisa ser construído.

**Foco principal:** Estado atual → Estado futuro. Tudo o mais (abordagem, riscos, limites) deriva dessa distinção.

**Ferramentas de mapeamento:** Quando o cenário tem fluxos de informação, dependências entre componentes ou atores, ou uma sequência de passos que muda entre hoje e o futuro — usar Mermaid para tornar a diferença visível. Diagramas são opcionais: usar quando ajudam a clarificar, não como ritual obrigatório.

**Model:** Observe corre antes de o projeto existir. Output → OBSERVE.md → /orbti:refine cria o projeto.

**Papel no framework:**

| Fase | Quem faz | Quando | Output |
|------|----------|--------|--------|
| **OBSERVE** | AI + usuário | antes de decidir | entendimento da realidade — o que existe, regras de negócio, o que está fora do escopo |
| **COCREATE** | AI especialista + usuário | antes de construir | decisões estratégicas + esboços técnicos que as sustentam |
| **RESEARCH** | AI | antes do REFINE | guardrails: o que não reabrir, onde aprofundar + as perguntas já respondidas |
</purpose>

<when_to_use>
- Antes de planejar qualquer trabalho — para mapear o que existe hoje e onde se quer chegar
- Quando a distância entre o cenário atual e o desejado não está clara
- Quando há múltiplos possíveis destinos e é preciso escolher um
- Quando o escopo é nebuloso — o observe força a clareza
</when_to_use>

<loop_context>
N/A - Pre-planning workflow. No project exists yet.
After discussion, routes to /orbti:refine which creates the project.
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/PROJECT.md
@.orbti/RUNBOOK.md (if exists — read silently, do not announce)
</required_reading>

<references>
@src/templates/OBSERVE.md (handoff format)
</references>

<process>

<step name="open_discussion">
**Abrir a conversa com o foco no mapeamento atual → futuro:**

Se $ARGUMENTS fornecido:
```
════════════════════════════════════════
OBSERVE
════════════════════════════════════════

Tópico: {arguments}

```

Se sem argumentos:
```
════════════════════════════════════════
OBSERVE
════════════════════════════════════════

```

Pergunta de abertura:

```
Como está hoje? O que você quer que mude?

Descreve o cenário atual — como funciona agora, quem usa, o que acontece.
Depois descreve onde quer chegar — o que melhora, o que passa a existir.
```

Aguardar resposta. O objetivo é capturar as duas pontas: **hoje** e **destino**.
</step>

<step name="explore_as_is">
**Aprofundar o cenário atual:**

Com base na resposta inicial, fazer perguntas para completar o mapa do hoje:

- Quem faz esse trabalho hoje? (pessoa, sistema, processo manual?)
- Como é acionado? (evento, horário, pedido?)
- O que quebra ou falta atualmente?
- Quais são as dores reais do cenário de hoje?

Store como `as_is`: atores, fluxo, problemas.

**Se o fluxo atual tem dependências ou múltiplos atores**, oferecer um diagrama Mermaid:
```
Quer que eu desenhe o fluxo atual em diagrama? (ajuda a confirmar o que entendi)
```
Se sim → gerar `flowchart LR` ou `sequenceDiagram` representando o estado atual.
Incluir no OBSERVE.md na seção "Como é hoje".
</step>

<step name="explore_goals">
**Mapear o estado futuro:**

```
Onde você quer chegar?

O que precisa existir que não existe hoje?
O que melhora ou some?
Como fica o fluxo depois que esse projeto existir?
```

- "Qual o resultado mais importante?"
- "Como seria um dia típico após isso pronto?"
- "O que uma pessoa percebe de diferente?"

Store como `goals` + `to_be`: o destino concreto, não objetivos vagos.

**Se o fluxo futuro é diferente do atual**, oferecer diagrama comparativo:
```
Quer um diagrama do estado futuro para comparar com o atual?
```
Se sim → gerar diagrama `to_be` com o mesmo estilo do `as_is`.
</step>

<step name="explore_approach">
**Discutir abordagem apenas depois de ter as duas pontas mapeadas:**

```
Como você quer chegar lá?

- Alguma tecnologia, padrão ou biblioteca que prefere usar ou evitar?
- Dependências em outros projetos ou times?
- Restrições de tempo, acesso ou recursos?
```

Store como `approach`. Se o usuário não souber — OK, o REFINE decide.
</step>

<step name="explore_boundaries">
**Define scope boundaries:**

Present suggested limits based on what was discussed, then ask:

```
Limites — o que esse projeto NÃO deve fazer?

Sugestões baseadas na discussão:
{derived_limits}

Ajusta, remove ou adiciona limites.
```

Wait for user response. Store as `boundaries` list.
</step>

<step name="explore_wormholes">
**Surface wormholes (hidden risks):**

Based on what was discussed, proactively identify risks the user may not have considered:

```
Riscos — riscos que não são óbvios:

{derived_wormholes}

Tem algo que eu não enxerguei? Algum que não se aplica?
```

Wait for user response. Store confirmed risks as `wormholes` list.
</step>

<step name="explore_solution_intent">
**Before synthesizing context, anchor the solution with three questions:**

```
Three quick questions to anchor the solution:

WHO uses this? (specific role and context — not "users" or "the team")
WHAT must they accomplish? (the primary verb/action — what are they trying to do?)
FEEL? (how should the solution behave — fast and silent? explicit and guided? dense? forgiving?)
```

Wait for answers. Store as `solution_intent`:
- `who`: specific person, role, and context
- `what`: primary action or outcome they need
- `feel`: personality of the solution — how it should behave in use

**Why this matters:** Without explicit intent, solutions default to generic patterns — the most common implementation seen in training, not the one that fits this specific problem. WHO/WHAT/FEEL forces the solution to be designed for a real context. UI, API design, error handling, and data modeling all change depending on these answers.

This populates OBSERVE.md and informs every phase that follows.

**Skip this step when work has no interface** — purely internal technical work: architecture refactors, security hardening, infrastructure changes, performance optimization. These have no interaction surface — COCREATE and REFINE's ACs handle intent adequately.
</step>

<step name="synthesize_context">
From the discussion, derive:

1. **Key goals** — synthesize main objectives
   - Confirm: "So the main goals are: {goals}. Sound right?"

2. **Approach notes** — capture technical direction and constraints

3. **What needs to be built** — concrete list of deliverables derived from goals and milestones

4. **Boundaries** — confirmed list of what this project must NOT do

5. **Wormholes** — confirmed risks and hidden complexities

6. **Open questions** — anything still unclear, items to decide during planning

Confirm with user before writing.
</step>

<step name="write_context">
Derive a project slug from the topic (kebab-case, lowercase, no accents).
Create folder `.orbti/projects/{slug}/` and write `OBSERVE.md` inside it.

Use the following OBSERVE.md structure:

```markdown
# CONTEXT — {Topic}

## Como é hoje
{descrição do cenário atual — atores, fluxo, o que existe, o que quebra}

{bloco mermaid aqui se diagrama foi construído}

## Para onde vamos
{descrição do cenário futuro — o que muda, o que passa a existir, como fica o fluxo}

{bloco mermaid aqui se diagrama foi construído}

## Goals
{objetivos derivados da diferença entre hoje e o destino — lista numerada}

## Marcos
{milestones as H3 sections}

## O que precisa ser construído
{concrete deliverables list}

## Approach
{technical direction and constraints}

## Limites (fora do escopo)
{boundaries — what this project must NOT do}

## Riscos
{numbered list of hidden risks and edge cases}

## Open Questions
{unresolved items for planning phase}

## Solution Intent
- **Who:** [specific person/role from discussion]
- **What:** [primary action or outcome they need]
- **Feel:** [how the solution should behave in use]

## Para a próxima fase
[Preenchido pelo research de alto nível — ponteiros de onde buscar na fase seguinte]

## Gestão de Projeto
- **Ferramenta:** [nome do PM tool, ex: Linear]
- **URL:** [link direto para o projeto na ferramenta]
```

If no PM tool was synced, omit the "Gestão de Projeto" section.

Display:
```
Observe saved to .orbti/projects/{slug}/OBSERVE.md

This file persists across /clear so you can take a break if needed.
/orbti:cocreate or /orbti:refine will pick it up automatically.
```
</step>

<step name="high_level_research">
**Após salvar o OBSERVE.md, executar research de alto nível no codebase para enriquecer o arquivo com o cenário atual concreto.**

Objetivo: entender o que já existe no produto hoje — no nível de negócio, sem descer para implementação.

**O que pesquisar:**

1. **Navegação / Itens de menu afetados**
   ```bash
   # Buscar rotas e itens de navegação no frontend
   grep -r "menu\|nav\|sidebar\|route" apps/crm-menos-juros/client/src/ --include="*.tsx" -l 2>/dev/null | head -10
   ```
   Identificar quais itens de menu existem hoje que serão afetados. Nível: nome do item, onde fica, o que faz.

2. **Domínios de negócio identificados**
   Ler `.orbti/codebase/OVERVIEW.md` se existir. Identificar quais domínios o projeto toca.
   ```bash
   cat .orbti/codebase/OVERVIEW.md 2>/dev/null | head -50
   ```

3. **Fluxos existentes que mudarão**
   Com base no OBSERVE.md recém criado (goals + as-is), identificar os fluxos existentes afetados. Buscar por nomes relacionados ao tópico.

4. **Entidades/conceitos de negócio relevantes**
   Não buscar por tipos ou classes — buscar por conceitos que o usuário mencionou na discussão.

**Nível de abstração:** produto e negócio. Não incluir nomes de arquivos, funções ou detalhes de implementação — isso é papel do refine.

**Adicionar ao OBSERVE.md as seguintes seções:**

```markdown
## Research de Alto Nível

### Domínios afetados
- [Domínio A] — [por quê é afetado]
- [Domínio B] — [por quê é afetado]

### Itens de navegação / menu afetados
- [Item] → [onde fica hoje, o que faz]

### Fluxos existentes que mudarão
- [Fluxo] → [como funciona hoje, o que muda]

### Entidades/conceitos de negócio relevantes
- [Entidade] → [o que representa]

### Perguntas abertas do research
O que não foi possível determinar neste nível de abstração — para o cocreate ou refine:
- [Pergunta] — [por que importa]

### Para a próxima fase
Ponteiros explícitos de *onde* o COCREATE (ou REFINE, se pular cocreate) deve buscar — não o que encontrar, mas onde olhar:
- `[path/módulo/tabela]` — [o que confirmar antes de decidir]
- `[path/módulo/tabela]` — [relevante para qual loop ou decisão]
```

**Regra da seção "Para a próxima fase":**
Lista de caminhos concretos, não de tópicos abstratos. "Ver `apps/api-menos-juros/src/v2/carteira/debito/`" é útil. "Verificar a arquitetura do backend" não é.

Se o codebase não está mapeado (sem OVERVIEW.md, sem COMPONENTS.md) → registrar nas perguntas abertas: "Mapeamento do codebase pendente — considerar `/orbti:map-codebase` antes do cocreate."

**Silencioso:** não anunciar cada busca ao usuário. Apenas exibir o resultado consolidado após inserir no OBSERVE.md.
</step>

<step name="suggest_diagrams">
**Sugerir diagramas Mermaid com base no que foi mapeado. O usuário decide quais construir.**

A partir do cenário atual e futuro capturados, analisar o que foi discutido e identificar quais diagramas ajudariam a tornar a diferença mais clara.

### Tipos de diagrama disponíveis

| Tipo | Quando usar |
|------|-------------|
| **Fluxo de informação** (`flowchart LR`) | Dados passam por múltiplos sistemas/atores — ex: CNAB vai de banco → processamento → parcela |
| **Sequência de passos** (`sequenceDiagram`) | Há uma ordem de eventos com troca de mensagens — ex: automação de remessa entre scheduler, FIDEM e banco |
| **Dependências** (`graph TD`) | Componentes ou serviços dependem uns dos outros — ex: qual módulo precisa de qual dado |
| **Estado atual vs futuro** (dois `flowchart` side-by-side) | O fluxo muda substancialmente entre hoje e o destino |
| **Mapa de atores** (`graph LR`) | Múltiplas pessoas, times ou sistemas interagem — deixa claro quem faz o quê |

### Como sugerir

Com base na discussão, propor apenas os diagramas que realmente ajudam — não todos. Exemplo:

```
Com base no que mapeamos, esses diagramas podem ajudar a visualizar:

[1] Fluxo de informação atual — como o CNAB chega hoje até a parcela
    (flowchart: Banco → Arquivo → Processamento → Parcela)

[2] Comparação atual vs futuro — o que muda no fluxo com a nova fonte de verdade
    (dois flowcharts lado a lado)

[3] Nenhum — o texto está claro o suficiente

Quais você quer construir?
```

Aguardar escolha. Construir os diagramas escolhidos e incluir no OBSERVE.md na seção correspondente (`## Como é hoje` ou `## Para onde vamos`).

**Se o contexto é simples e os diagramas não agregariam** (ex: uma única feature sem múltiplos atores) → pular este step.
</step>

<step name="pm_sync">
**Check for project management MCP and offer sync:**

Scan available MCP tools for known project management integrations:
- Linear: tools named `mcp__plugin_linear_linear__*` or `mcp__linear__*`
- GitHub Issues: tools named `mcp__github__*`
- Jira: tools named `mcp__jira__*`
- Notion: tools named `mcp__notion__*`
- Any other PM tool MCPs connected

If one or more PM MCPs are detected, use AskUserQuestion:
```
question: "Quer registrar esse projeto em {tool_name}?"
header: "Gestão de projeto"
options:
  - label: "Adicionar"
    description: "Criar/atualizar o projeto em {tool_name} — vou pedir a URL ou o nome"
  - label: "Pesquisar"
    description: "Buscar automaticamente pelo título do projeto em {tool_name}"
  - label: "Não agora"
    description: "Continuar sem registrar no {tool_name}"
```

**Route:**
- "Adicionar" → ask user for the project URL or name in {tool_name}, then create/update with goals + milestones + limits. Save URL in OBSERVE.md under "Gestão de Projeto".
- "Pesquisar" → use the PM MCP to search by the project topic/slug, present matches, confirm with user, then update. Save URL in OBSERVE.md.
- "Não agora" → skip, omit "Gestão de Projeto" section from OBSERVE.md.

If no PM MCP is detected → skip this step silently.
</step>

<step name="handoff">
Display summary:
```
════════════════════════════════════════
OBSERVE COMPLETE
════════════════════════════════════════

Goals: {goal_count}
Marcos: {milestone_count}
Riscos identificados: {wormhole_count}
Status: Pronto para planejamento
```

**Check design config (special flow):**

```bash
grep -A2 "^design:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Se `design: enabled: true` E o OBSERVE.md contém interfaces de usuário (telas, dashboards, formulários):

```
Esse projeto tem interfaces a construir. Quer rodar /orbti:design para criar protótipos antes de planejar?

Os protótipos gerados serão passados automaticamente para o /orbti:refine como referência visual.
```

Usar AskUserQuestion:
```
question: "Criar protótipos das interfaces antes de planejar?"
header: "Design especial flow"
options:
  - label: "Sim, criar protótipos"
    description: "Explorar design e gerar protótipos → /orbti:design → /orbti:refine"
  - label: "Não, ir para planejamento"
    description: "Pular design e ir direto ao planejamento → /orbti:refine"
```

**Route design flow:**
- "Sim" → run `/orbti:cocreate {slug}` (design mode — produz DESIGN.md + system.md) → depois `/orbti:refine {slug}`
- "Não" → seguir para o routing padrão abaixo

Se `design: enabled: false` ou nenhuma interface no OBSERVE.md → pular silenciosamente.

**Routing padrão:**

Then use AskUserQuestion:
```
question: "O que fazer agora?"
header: "Próximo passo"
options:
  - label: "Refine"
    description: "Quebrar goals em fases e tarefas → /orbti:refine"
  - label: "Cocreation"
    description: "Explorar opções técnicas antes de planejar → /orbti:cocreate"
  - label: "New Observe"
    description: "Iniciar um novo observe para outro projeto → /orbti:observe"
```

**Route:**
- "Refine" → run `/orbti:refine {slug}`
- "Cocreation" → run `/orbti:cocreate {slug}`
- "New Observe" → run `/orbti:observe`
</step>

</process>

<output>
- .orbti/projects/{slug}/OBSERVE.md created (handoff file for /orbti:refine)
- Goals, boundaries, wormholes and approach articulated
- Project management tool updated (if MCP available and user confirmed)
</output>

<success_criteria>
- [ ] Goals explored (user-driven)
- [ ] As-is process captured
- [ ] Approach discussed
- [ ] Boundaries defined
- [ ] Wormholes surfaced
- [ ] Context synthesized and confirmed
- [ ] OBSERVE.md written to .orbti/projects/{slug}/
- [ ] Mermaid diagrams built (if user chose any)
- [ ] PM tool synced (if applicable)
- [ ] Design special flow offered (if design: enabled and interfaces present)
- [ ] Clear handoff to /orbti:refine
</success_criteria>

<anti_patterns>
**Asking abstract questions first:**
DON'T: "What's the scope of this project?"
DO: "What do you want to accomplish?"

**Assuming approach before goals:**
DON'T: "What libraries will you use?"
DO: Derive approach from goals discussed.

**Requiring a project to exist:**
DON'T: Validate against ROADMAP.md or require a project number.
DO: This runs before the project exists. /orbti:refine creates it.

**Not persisting context:**
DON'T: End discussion without writing OBSERVE.md
DO: Always write the file so /clear doesn't lose progress.

**Skipping wormholes:**
DON'T: Leave risks implicit or unspoken.
DO: Proactively derive risks from the discussion and surface them.

**Skipping boundaries:**
DON'T: Let scope be open-ended.
DO: Always define what this project must NOT do.
</anti_patterns>

<error_handling>
**User unsure what to accomplish:**
- Reference PROJECT.md for overall goals
- Offer: "We can start broad and narrow down"

**Scope too large:**
- Note it during planning: "This sounds like multiple projects — we can split in /orbti:refine"

**User wants to skip discussion:**
- Route directly to /orbti:refine
- Note: "Going straight to planning — no discussion context will be available"

**No PM MCP detected:**
- Skip pm_sync step silently — do not mention it.
</error_handling>

<workflow-dev>
# Decisões de Design — OBSERVE workflow

## Refoco: Estado atual → Estado futuro (2026-04-08)

**Decisão:** O observe foi refocado para ter como objetivo principal mapear o que existe hoje e onde se quer chegar — não estruturar requisitos, não planejar.

**Rationale:** O observe estava se tornando um planejamento antecipado disfarçado. O usuário pedia para descrever o cenário e já entrava em arquitetura, tecnologias e tasks. O refoco força: primeiro as duas pontas (hoje e destino), depois tudo o mais (abordagem, riscos, limites) deriva da diferença entre elas.

**Como aplicar:** A pergunta de abertura é sempre "Como está hoje? O que você quer que mude?" — não "Qual é o objetivo?". A abordagem técnica só entra APÓS as duas pontas estarem mapeadas.

---

## Mermaid: opcional e interativo (2026-04-08)

**Decisão:** Diagramas Mermaid são sugeridos interativamente após o mapeamento textual. O usuário decide quais construir ou nenhum. Não são obrigatórios.

**Rationale:** Diagramas forçados viram ritual. Alguns contextos (uma única feature simples, um processo linear sem ramificações) não precisam de diagrama — o texto já é claro o suficiente. Outros têm múltiplos atores, dependências cruzadas ou fluxos que mudam substancialmente entre hoje e o futuro — nesses casos o diagrama é o meio mais eficiente de confirmar o entendimento.

**Como aplicar:** Depois de escrever o OBSERVE.md, o `suggest_diagrams` step analisa o que foi discutido e propõe os diagramas que agregariam valor — máximo 3 sugestões específicas + opção "nenhum". O usuário escolhe. Qualquer diagrama construído vai para o OBSERVE.md na seção correspondente (`## Como é hoje` ou `## Para onde vamos`).

---

## Design special flow: orbti:design → protótipos → refine (2026-04-08)

**Decisão:** Quando `design: enabled: true` no config.md do projeto e o OBSERVE.md identifica interfaces, o observe oferece rodar o orbti:design antes de planejar. Os protótipos gerados servem de referência visual para o refine.

**Rationale:** Planejar sem ver a interface gera REFINE-FRONT com ACs genéricas demais ("deve exibir os dados", "deve ter filtro"). Com um protótipo como âncora, o refine consegue ser específico: "deve implementar a tabela densa com a estrutura X conforme protótipo". O design precede o planejamento porque informa o que precisa ser construído e como.

**Como aplicar:** O check é silencioso — só aparece se `design: enabled: true` E há interfaces no OBSERVE.md. O route vai para `/orbti:cocreate {slug}` em design mode (produz DESIGN.md + system.md + protótipos via orbti:design) e depois para `/orbti:refine`. Se o usuário recusar, segue o routing padrão sem mencionar o design de novo.

---

## OBSERVE.md: "Como é hoje" antes dos Goals (2026-04-08)

**Decisão:** O template do OBSERVE.md foi reorganizado para começar com "Como é hoje" e "Para onde vamos" — antes dos Goals, Marcos e demais seções.

**Rationale:** O OBSERVE.md é o artefato de handoff do observe. Se a ordem reflete a sequência de raciocínio do observe (atual → futuro → diferença → o que construir), o refine consegue ler o OBSERVE.md de forma linear e entender a motivação por trás de cada goal. Goals sem o contexto "de onde viemos" ficam flutuando.
</workflow-dev>
