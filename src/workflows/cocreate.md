<purpose>
Fase opcional entre o observe e o refine. Materializa a visão desejada em um plano estratégico por specialist (F, B, T, A) e por loop. Aprofunda o research do OBSERVE.md. Não é o como técnico — é o como estratégico.

**Pré-requisito obrigatório:** OBSERVE.md deve existir na pasta do projeto. Se não existir, bloquear e orientar `/orbti:observe` primeiro.
</purpose>

<when_to_use>
- Após o observe, quando a solução ainda não está materializada o suficiente para o refine
- Quando há múltiplas possibilidades de como construir e precisa-se definir a estratégia
- Quando o projeto tem múltiplos specialists (F+B+T+A) e precisa organizar a ordem dos loops
- Quando o usuário quer pensar no design antes de planejar
- Opcional: o usuário pode ir direto do observe para o refine sem passar pelo cocreate
</when_to_use>

<loop_context>
Pre-planning workflow. Runs after observe, before refine.
Output → COCREATE.md → /orbti:refine reads it as strategic input.
</loop_context>

<required_reading>
@.orbti/projects/{slug}/OBSERVE.md
@.orbti/STATE.md
</required_reading>

<references>
@src/templates/COCREATE.md
</references>

<process>

<step name="validate_observe">
**Primeiro: verificar que o observe foi realizado.**

```bash
ls .orbti/projects/{slug}/OBSERVE.md 2>/dev/null || echo "NOT_FOUND"
```

Se OBSERVE.md não existe:
```
════════════════════════════════════════
⛔ COCREATE BLOQUEADO
════════════════════════════════════════
OBSERVE.md não encontrado em .orbti/projects/{slug}/

O cocreate precisa do observe para funcionar.
Execute primeiro: /orbti:observe {slug}
════════════════════════════════════════
```
→ PARAR.

Se existe → carregar OBSERVE.md e COCREATE.md anterior (se existir) → prosseguir.
</step>

<step name="open_hmw">
**Abrir a cocriação com 5 perguntas HMW estratégicas (How Might We?).**

Derivar as 5 perguntas do OBSERVE.md — **antes do research**. As respostas guiam o que pesquisar.

**Escopo: HMW estratégico apenas — "o que e por quê".**
NÃO incluir HMW técnico ("reutilizar fila X ou criar nova?", "coluna ou JSONB?") — esses pertencem ao REFINE, após o research técnico.

Exibir:
```
════════════════════════════════════════
COCREATE — {slug}
════════════════════════════════════════

Antes de pesquisar, 5 perguntas para orientar a solução:

HMW 1: Como poderíamos {X} para que {usuário} consiga {resultado}?
HMW 2: Como poderíamos {X} sem {trade-off indesejado}?
HMW 3: Como poderíamos {X} mesmo que {restrição do OBSERVE.md}?
HMW 4: Como poderíamos garantir {Y} considerando {risco identificado}?
HMW 5: Como poderíamos {X} de forma que {feel do Solution Intent}?

────────────────────────────────────────
Responda as que fazem sentido, ajuste ou descarte as que não se aplicam.
As respostas vão guiar o que pesquisar a seguir.
```

Aguardar respostas. Usar as respostas para **direcionar o research** do próximo step.

**Critérios para boas perguntas HMW estratégicas:**
- Específicas ao projeto (não "como poderíamos melhorar a UX?")
- Abrem possibilidades sem restringir implementação
- Cobrem tensões identificadas no observe (riscos, prioridades, trade-offs de produto)
- Uma por ângulo: usuário, produto, restrição, risco, feel
- **Não** perguntam sobre tecnologia, biblioteca ou pattern específico
</step>

<step name="deepen_research">
**Research estratégico — constrói sobre o OBSERVE, não repete.**

**Regra fundamental:** ler o que o OBSERVE.md já descobriu e partir daí. Não re-escanear menus, re-mapear fluxos ou re-descrever o sistema atual — isso já está no OBSERVE.

**O que pesquisar aqui:**
Guiado pelas respostas HMW do step anterior. Cada resposta HMW é uma pergunta aberta — o research responde com o que existe no codebase/produto que informa a decisão.

**Nível deste research:** estratégico/UX — não técnico.
- Como as telas funcionarão? Que fluxos o usuário percorre, o que vê, o que faz
- Quais partes do sistema precisam existir antes de outras? (dependências de construção)
- O que é mais simples de construir primeiro? (risco vs valor por loop)

**Ainda não inclui:** arquivos específicos, funções, schemas, hooks, migrations — isso é o REFINE.

**Progressividade do research por fase:**
```
OBSERVE   → negócio: domínios afetados, menus, fluxos existentes
COCREATE  → estratégico/UX: como funciona para o usuário, ordem de construção
REFINE    → técnico: arquivos específicos, hooks, schemas, endpoints
```

Adicionar ao COCREATE.md uma seção "## Research Aprofundado" com o que é **novo** em relação ao OBSERVE.
</step>

<step name="build_plan">
**Construir o plano por specialist e por loop.**

Com base no OBSERVE.md + respostas HMW + research aprofundado, produzir o plano:

```markdown
## Plano por Loop e Specialist

### Loop 1 — [Descrição do objetivo]

#### Front (F)
O que será construído ou alterado neste loop:
- Tela/componente [X]: [como funciona, interações, o que mostra]
- Navegação: [alterações no menu ou rotas]
- [O que reutilizar vs o que criar novo]

#### Back (B)
O que será alterado ou construído:
- [Endpoint / regra / migration] — [descrição]
- [O que muda vs o que é novo]

#### Agents (A) [omitir se não aplicável]
- [Automação] → [trigger, ação, resultado esperado]

#### Tests (T)
Cenários de validação para este loop:
- [Cenário]: [resultado esperado]

### Loop 2 — [Descrição]
[mesma estrutura]
```

**Regras do plano:**
- Loop 1 deve ser o de menor risco e maior valor — fundação para os demais
- Cada loop deve ser independente e testável
- Front e Back do mesmo loop rodam em paralelo no BUILD
- Test só aparece no plano se `test: enabled: true` no config.md

**Sobre as telas (Front):** especificar como funcionam — não só "criar tela X" mas "tela X exibe Y, filtra por Z, ação primária é W, estado vazio é V".
</step>

<step name="write_cocreate">
**Salvar COCREATE.md na pasta do projeto.**

Path: `.orbti/projects/{slug}/COCREATE.md`

```markdown
---
project: {slug}
created: YYYY-MM-DD
observe: OBSERVE.md
---

# COCREATE — {Tópico}

## Visão Materializada
[Síntese da visão — mais concreta que o observe, menos técnica que o refine]

## HMW — Perguntas de Cocriação
1. Como poderíamos...
2. Como poderíamos...
3. Como poderíamos...
4. Como poderíamos...
5. Como poderíamos...

## Research Aprofundado
[Descobertas além do OBSERVE.md — nível de UX e planejamento estratégico]

## Plano por Loop e Specialist

### Loop 1 — [Descrição]
#### Front (F)
...
#### Back (B)
...
#### Agents (A) [se aplicável]
...
#### Tests (T)
...

### Loop 2 — [Descrição]
...

## Protótipos [seção adicionada após /orbti:design, se executado]
- [Tela X]: [tool — arquivo/artboard/URL]

## Decisões de Cocriação
[Escolhas estratégicas — com rationale]

## Handoff para Refine

> Leitura obrigatória do /orbti:refine. O refine não re-descobre o que o cocreate já resolveu.

### Loop 1 — [Descrição]

**Leia antes de criar tasks:**
- `[arquivo]` — [o que verificar]

**HMW técnico (abrir no início do refine):**
- [Decisão]: [contexto e opções conhecidas]

**Riscos:**
- [Risco]: [como mitigar]

**Protótipos:**
- [Tela]: [referência]

---

### Loop 2 — [Descrição]
[mesma estrutura]
```

**Como preencher o Handoff:**

O handoff é o documento mais importante do COCREATE para a continuidade. Regras:

1. **"Leia antes de criar tasks"** — arquivos específicos que o refine PRECISA ler para não tomar decisão errada. Não listar tudo — só o que muda o design da solução se não for lido.

2. **"HMW técnico"** — decisões que o cocreate identificou mas NÃO resolve (porque exigem o codebase). São as perguntas que o refine vai abrir upfront. Incluir contexto suficiente para a pergunta fazer sentido sem ler todo o COCREATE.md.

3. **"Riscos"** — apenas os que impactam como as tasks devem ser escritas. Não lista de preocupações gerais.

4. **"Protótipos"** — referências exatas. O refine usa isso para ancorar as tasks de frontend.

**Objetivo:** o refine lê só essa seção e já sabe onde começar — sem precisar parsear o COCREATE.md inteiro.

Display:
```
COCREATE salvo em .orbti/projects/{slug}/COCREATE.md
```
</step>

<step name="offer_design">
**Verificar se design está ativo e oferecer prototipagem.**

```bash
grep -A2 "^design:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Se `design: enabled: true` E o plano tem interfaces de usuário (telas, dashboards, formulários):

Usar AskUserQuestion:
```
question: "Quer criar protótipos das interfaces antes de planejar?"
header: "Design — prototipagem"
options:
  - label: "Sim, prototipar"
    description: "Gerar protótipos via /orbti:design → COCREATE.md será atualizado com as referências"
  - label: "Não, ir para o refine"
    description: "Pular design e ir direto para o planejamento técnico"
```

**Se "Sim":**
1. Invocar `/orbti:design {slug}` com o plano de telas do COCREATE.md como input
2. Após retornar, adicionar seção "## Protótipos" no COCREATE.md com as referências criadas
3. Prosseguir para handoff

**Se "Não":** prosseguir diretamente para handoff.

Se `design: enabled: false` → pular silenciosamente.
</step>

<step name="handoff">
Display:
```
════════════════════════════════════════
COCREATE COMPLETE
════════════════════════════════════════

HMW: 5 perguntas de cocriação
Loops: {N} loops planejados
Specialists: {lista de specialists por loop}
Design: {protótipos criados | não executado}

────────────────────────────────────────
▶ NEXT: /orbti:refine {slug}
  Planejamento técnico do loop 1
────────────────────────────────────────
```
</step>

</process>

<output>
- .orbti/projects/{slug}/COCREATE.md (plano estratégico por loop e specialist)
- COCREATE.md atualizado com protótipos (se design executado)
</output>

<success_criteria>
- [ ] OBSERVE.md verificado (pré-requisito)
- [ ] 5 perguntas HMW derivadas e respondidas
- [ ] Research aprofundado além do OBSERVE.md
- [ ] Plano por loop e specialist com telas especificadas
- [ ] Design oferecido (se config habilitada)
- [ ] COCREATE.md salvo
- [ ] Handoff claro para /orbti:refine
</success_criteria>

<anti_patterns>
**Executar sem OBSERVE.md:**
DON'T: Iniciar cocreate sem verificar o observe.
DO: Bloquear e redirecionar para /orbti:observe.

**HMW genéricas:**
DON'T: "Como poderíamos melhorar a experiência?"
DO: "Como poderíamos mostrar o status de cada parcela sem sobrecarregar a tela de detalhes?"

**Entrar em detalhes técnicos:**
DON'T: "Usar o hook useRetornoDavos com parâmetro opcional id"
DO: "Tela de conciliação mostra lista de remessas, filtrável por status e data"

**Pular telas:**
DON'T: "Front: construir dashboard de eficiência"
DO: "Front: dashboard exibe 3 cards (remessas pendentes, integradas, rejeitadas), tabela filtrável por data e status, ação 'Remeter' disponível para itens pendentes"
</anti_patterns>
</output>