---
name: orbti:cocreate
description: Research technical options and make architecture/library decisions before planning. Produces COCREATE.md with recommendation and confidence level. For design decisions, use /orbti:refine which supports Tech and Design modes.
argument-hint: "<project or topic>"
allowed-tools: [Read, Bash, Glob, Grep, WebSearch, WebFetch, Task, AskUserQuestion, Skill]
---

<objective>
Execute technical discovery to inform planning decisions.

**Output:** COCREATE.md with architecture/library findings, recommendation, and confidence level.

**When to use:** Before planning a project with technical unknowns — architecture, DDD, libraries, integrations.

**Distinct from /orbti:research:** Research gathers information. Cocreate makes decisions.
**Distinct from /orbti:refine:** Refine plans the work. Cocreate resolves technical unknowns before planning.

**Not part of the main loop** — run explicitly when there are genuine technical unknowns to resolve.
</objective>

<model>opus</model>

<execution_context>
@./.claude/orbti-framework/workflows/cocreate.md
@./.claude/orbti-framework/templates/COCREATE.md
@./.claude/orbti-framework/references/model-routing.md
@./.claude/skills/ddd/SKILL.md
</execution_context>

<context>
$ARGUMENTS (topic or question)

@.orbti/STATE.md
@.orbti/PROJECT.md
</context>

<ddd_architecture>
Quando o tópico envolver organização de arquivos, estrutura de módulos, nova feature na api-menos-juros, ou qualquer decisão arquitetural, aplicar os princípios DDD do SKILL.md carregado acima.

## Passo obrigatório: classificar cada conceito da feature

Antes de propor arquivos, perguntar e responder para cada conceito levantado:

| Pergunta | Resposta → Tipo |
|----------|-----------------|
| "Quem inicia a ação?" | → **Use Case** (`registrar-proposta.use-case.ts`) |
| "O que aconteceu?" | → **Domain Event** (`proposta-registrada.event.ts`) |
| "O que é / tem identidade?" | → **Entity** (`proposta.entity.ts`) |
| "Descreve um valor sem identidade?" | → **Value Object** (`cpf.value-object.ts`) |
| "Quem decide, mas não pertence a nenhuma entidade?" | → **Domain Service** |
| "Outros contextos precisam saber?" | → **Integration Event** via BullMQ |
| "Como acessar/persistir?" | → **Repository** (port + implementação) |

**Durante a cocriação**, para cada conceito que o usuário mencionar, aplicar o fluxo:
1. Nomear o conceito com a linguagem ubíqua do negócio
2. Classificar com as perguntas acima
3. Definir camada (`domain/`, `application/`, `infrastructure/`)
4. Propor nome de arquivo seguindo convenção

**Checklist estrutural:**
- Identificar o Bounded Context afetado (`comercial/`, `mesa-analise/`, `carteira/`, `core/`)
- Verificar se precisa de Domain Events (intra-contexto) ou Integration Events (BullMQ, cross-contexto)
- Verificar dependências entre camadas (domain sem imports de infra/framework)

**Quando listar arquivos no COCREATE.md**, incluir colunas: `Arquivo | Tipo DDD | Camada | Justificativa`.
</ddd_architecture>

<process>

## Passo 0 — Detectar interfaces e verificar config de design

**Execute ANTES de qualquer discovery.**

### 0a. Ler contexto do projeto

Localizar o CONTEXT.md do projeto atual (via STATE.md ou $ARGUMENTS):
```bash
cat .orbti/projects/{slug}/CONTEXT.md 2>/dev/null
```

### 0b. Identificar se há interfaces a construir

Analisar a seção "O que precisa ser construído" do CONTEXT.md. Identificar itens que são interfaces de usuário:
- Telas, dashboards, painéis, modais, formulários, páginas
- Qualquer entregável com superfície visual interativa

**Contar quantas interfaces foram identificadas.**

Se **nenhuma interface** encontrada → seguir direto para Tech mode (Passo 2).

### 0c. Criar DESIGN.md

**Sempre criar** quando há interfaces — seja uma ou várias. O conteúdo varia conforme a quantidade.

#### Interface única

```markdown
# DESIGN — {nome do projeto}

## Interface: {Nome da Interface}

**Propósito:** {objetivo principal em uma frase}

## Solution Intent

- **Who:** {de CONTEXT.md → Solution Intent → Who}
- **What:** {de CONTEXT.md → Solution Intent → What}
- **Feel:** {de CONTEXT.md → Solution Intent → Feel}

## Features a construir

- {feature 1 derivada do CONTEXT.md}
- {feature 2}
- {feature 3}

## Contexto de uso

{onde aparece no fluxo do produto / quem usa e quando / frequência de uso}

## Dados e estados

{quais dados a interface exibe ou manipula — listas, métricas, formulários, status, etc.}

## Notas de design

{constraints, riscos visuais, plataforma, referências relevantes do CONTEXT.md}
```

#### Múltiplas interfaces (2+)

```markdown
# DESIGN — {nome do projeto}

*{N} interfaces a construir*

---

## 1. {Nome da Interface}

**Propósito:** {objetivo principal em uma frase}

**Features:**
- {feature 1 derivada do CONTEXT.md}
- {feature 2}
- {feature 3}

**Contexto de uso:** {onde aparece no fluxo / quem usa e quando}

**Dados e estados:** {o que exibe ou manipula}

---

## 2. {Nome da Interface}

**Propósito:** {objetivo principal em uma frase}

**Features:**
- {feature 1}
- {feature 2}

**Contexto de uso:** {onde aparece no fluxo / quem usa e quando}

**Dados e estados:** {o que exibe ou manipula}

---

[repetir para cada interface]

## Solution Intent (compartilhado)

- **Who:** {de CONTEXT.md → Solution Intent → Who}
- **What:** {de CONTEXT.md → Solution Intent → What}
- **Feel:** {de CONTEXT.md → Solution Intent → Feel}

## Notas de design

{constraints transversais, riscos visuais, plataforma, referências do CONTEXT.md}
```

Exibir ao usuário:
```
DESIGN.md criado — {1 interface | N interfaces}
→ .orbti/projects/{slug}/DESIGN.md
```

### 0e. Perguntar roteamento Design/Tech

Com design config ativo, usar AskUserQuestion:
```
question: "Qual tipo de cocriação executar?"
header: "Cocriação"
options:
  - label: "Tech"
    description: "Descoberta técnica — arquitetura, DDD, bibliotecas → COCREATE.md"
  - label: "Design"
    description: "Exploração de design — intenção, domínio visual, direção → orbti-design"
  - label: "Ambos"
    description: "Tech primeiro, depois Design"
```

---

## Passo 1 — Design mode: exploração de intent via orbti-design

**Executar quando rota for Design ou Ambos.**

1. Carregar a skill orbti-design:
   ```
   Skill("orbti-design")
   ```

2. Usar o DESIGN.md como input principal (já foi criado no Passo 0d):
   - **WHO/WHAT/FEEL** → `Solution Intent` do DESIGN.md (não pedir de novo ao usuário)
   - **Features** → seções de features por interface
   - **Dados e estados** → o que a interface exibe/manipula
   - **Notas de design** → constraints e referências

3. Executar o fluxo "Intent First" + "Product Domain Exploration" do orbti-design SKILL.md usando o DESIGN.md como âncora:
   - Responder WHO/WHAT/FEEL com os dados do DESIGN.md (já extraídos do CONTEXT.md)
   - Executar domain exploration (Domain, Color world, Signature, Defaults)
   - Propor direção e confirmar com o usuário

4. **Interface única:** focar toda a exploração na interface descrita no DESIGN.md.
   **Múltiplas interfaces:** propor uma direção de design unificada (mesma linguagem visual para o conjunto), anotando variações específicas por interface se necessário.

5. Após confirmação de direção, oferecer salvar em `.orbti-design/system.md` seguindo o padrão do orbti-design SKILL.md.

---

## Passo 2 — Tech mode: discovery técnico

**Executar quando rota for Tech ou Ambos.**

**Follow workflow: @./.claude/orbti-framework/workflows/cocreate.md**

The workflow implements — nesta ordem obrigatória:
1. Verificar OBSERVE.md (pré-requisito)
2. **5 HMW estratégicos** — derivados do OBSERVE/CONTEXT.md, ANTES de qualquer research
   - HMW estratégico: "o que e por quê" — produto, prioridade, restrição, risco, feel
   - **NÃO inclui** HMW técnico ("reutilizar fila ou criar nova?") — esse é papel do REFINE
   - Aguardar respostas → respostas guiam o research
3. Research estratégico (subagents) — constrói sobre o OBSERVE, não repete o que já foi descoberto
   - **If architectural topic:** apply DDD checklist from `<ddd_architecture>` above
4. Plano por loop e specialist
5. Create COCREATE.md (include file map with DDD layers when applicable)
6. Route to planning when complete

---

## Passo 3 — Finalizar e rotear

Após concluir os modes selecionados, exibir:

```
════════════════════════════════════════
COCREATE COMPLETE
════════════════════════════════════════

Modo: [Tech | Design | Ambos]
{Se Tech:}  COCREATE.md → .orbti/projects/{slug}/COCREATE.md
{Se Design:} DESIGN.md  → .orbti/projects/{slug}/DESIGN.md
{Se Design e system.md salvo:} system.md → .orbti-design/system.md

────────────────────────────────────────
▶ NEXT: /orbti:refine {slug}
────────────────────────────────────────
```

</process>

<success_criteria>
**Geral:**
- [ ] Interfaces detectadas (ou ausência confirmada)
- [ ] DESIGN.md criado se há interfaces (antes de qualquer discovery)

**Tech mode:**
- [ ] Depth determinado
- [ ] Unknowns identificados
- [ ] Opções pesquisadas com fontes
- [ ] COCREATE.md criado
- [ ] Recomendação com nível de confiança

**Pronto para /orbti:refine**
</success_criteria>
