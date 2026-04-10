# COCREATE — Fase de Cocriação Estratégica

**Obrigatória?** Não — fase opcional entre o OBSERVE e o REFINE.

O COCREATE é a fase do **como estratégico**. Não é planejamento técnico — é a materialização da visão em um plano concreto por specialist e por loop. Responde "o que construir, em que ordem" antes de responder "como construir tecnicamente" (que é papel do REFINE).

**Pré-requisito:** OBSERVE.md deve existir na pasta do projeto.

---

## Papel no Framework

| Fase | Quem faz | Quando | Output |
|------|----------|--------|--------|
| **OBSERVE** | AI + usuário | antes de decidir | entendimento da realidade — o que existe, regras de negócio, o que está fora do escopo |
| **COCREATE** | AI especialista + usuário | antes de construir | decisões estratégicas + esboços técnicos que as sustentam |
| **RESEARCH** | AI | antes do REFINE | guardrails: o que não reabrir, onde aprofundar + as perguntas já respondidas |

---

## O que o COCREATE faz

1. **5 perguntas HMW** — abre a cocriação com perguntas "How Might We?" específicas ao projeto
2. **Aprofunda o research** — pesquisa em nível estratégico/UX, além do que o OBSERVE mapeou
3. **Produz o plano** — especifica o que construir por specialist (F, B, T, A) e por loop
4. **Oferece design** — se configurado, chama o orbti:design para prototipar antes de planejar
5. **Salva COCREATE.md** — handoff estruturado para o REFINE

---

## Quando usar

- Quando a solução tem múltiplas telas complexas e é preciso definir a ordem dos loops
- Quando há dúvida sobre o que construir antes (dependências entre loops)
- Quando o projeto envolve múltiplos specialists (F + B + A) e é preciso coordenar
- Quando o usuário quer pensar no design antes de planejar o código
- **Não é necessário** para projetos simples ou features pequenas — vá direto ao REFINE

---

## Como usar

```bash
/orbti:cocreate {slug}

# Com opções:
/orbti:cocreate {slug} --design       # força etapa de design
/orbti:cocreate {slug} --skip-design  # pula design mesmo se configurado
```

---

## As 5 Perguntas HMW — Estratégicas

O COCREATE abre 5 perguntas HMW **antes do research** — as respostas guiam o que pesquisar.

**Tipo: estratégico.** Perguntas sobre "o quê e por quê" — produto, prioridade, restrição, risco, feel.
**Não inclui** perguntas técnicas de implementação ("reutilizar X ou criar Y?") — essas são papel do REFINE.

**Por que antes do research?**
- As respostas tornam o research focado — você pesquisa o que importa, não tudo
- Sem HMW, o research tende a ser completo mas sem direção
- HMW técnico sem pesquisar o codebase seria genérico e inútil — por isso fica no REFINE

**Formato típico:**
```
HMW 1: Como poderíamos {X} para que {usuário} consiga {resultado}?
HMW 2: Como poderíamos {X} sem {trade-off indesejado}?
HMW 3: Como poderíamos {X} mesmo que {restrição identificada}?
HMW 4: Como poderíamos garantir {Y} considerando {risco do OBSERVE}?
HMW 5: Como poderíamos {X} de forma que {feel desejado}?
```

**O que NÃO é HMW estratégico:**
- "Como implementar X usando Y ou Z?" → técnico, vai para REFINE
- "Reutilizar fila existente ou criar nova?" → técnico, vai para REFINE
- "Usar coluna nova ou campo JSONB?" → técnico, vai para REFINE

---

## Como o COCREATE alimenta o RESEARCH.md

O COCREATE aprofunda a segunda camada do research progressivo:

```
OBSERVE → "quais domínios são afetados?" → research de negócio
COCREATE → "como a tela X funciona para o usuário?" → research estratégico/UX
         → "o que precisa existir antes de quê?" → dependências de construção
         → adiciona ao RESEARCH.md: "## Nível Estratégico"
```

---

## O Plano por Specialist e Loop

O output principal do COCREATE é o plano organizado por loops e specialists:

```markdown
## Loop 1 — Fundação (menor risco, maior valor)

### Front (F)
- Tela de listagem de remessas: exibe tabela paginada, filtrável por status e data,
  ação primária "Remeter" disponível para itens pendentes, estado vazio mostra
  "Nenhuma remessa pendente" com CTA para criar
- Menu: adicionar "Remessas" sob "Carteira"

### Back (B)
- Endpoint GET /remessas: retorna lista paginada com filtros de status e data
- Migration: criar tabela remessas com campos básicos

### Tests (T)
- Cenário: listagem vazia exibe estado correto
- Cenário: filtro por status funciona

## Loop 2 — Detalhamento
...
```

**Regra importante:** especificar como as telas **funcionam**, não só que existem. "Tela X exibe Y, com ação Z, estado vazio W" — não apenas "criar tela X".

---

## Design (Special Flow Opcional)

Se `design: enabled: true` no `config.md` e o orbti-design estiver instalado, o COCREATE oferece criar protótipos antes de planejar:

```
Quer prototipar as interfaces antes do REFINE?
[1] Sim — /orbti:design gera protótipos → COCREATE.md é atualizado com as referências
[2] Não — ir direto para /orbti:refine
```

Os protótipos servem como âncora para o REFINE — em vez de "criar tabela de remessas", o REFINE diz "implementar tabela conforme protótipo X, usando `<Table>` de shadcn com as colunas A, B, C".

---

## Estrutura de arquivos criada

```
.orbti/projects/{slug}/
├── OBSERVE.md     ← input (deve existir)
└── COCREATE.md    ← criado pelo cocreate
```

---

## O Handoff para Refine — Seção mais importante do COCREATE

O COCREATE termina com um handoff estruturado por loop. **Este é o documento que o refine lê primeiro** — antes de qualquer research técnico.

**Objetivo:** o refine não re-descobre o que o cocreate já resolveu. Lê o handoff, extrai contexto, começa de onde o cocreate parou.

```markdown
## Handoff para Refine

### Loop 1 — [Descrição]

**Leia antes de criar tasks:**
- `path/arquivo.ts` — [o que verificar — muda o design se não for lido]

**HMW técnico (abrir no início do refine):**
- [Decisão a tomar]: [contexto + opções conhecidas]

**Riscos:**
- [Risco]: [como mitigar nas tasks]

**Protótipos:**
- [Tela X]: [referência exata — .pen / artboard / URL]
```

**O que o handoff contém:**
- Arquivos que DEVEM ser lidos antes das tasks (só os que mudam o design da solução)
- HMW técnico upfront — decisões que o cocreate identificou mas não resolve (exigem codebase)
- Riscos que impactam como as tasks são escritas
- Referências de protótipos (para tasks de frontend)

**Redução de tokens:** o refine lê só o handoff, não o COCREATE.md inteiro.

---

## O que vai para o REFINE

O REFINE usa o handoff como entrada principal. Para cada loop, transforma o "o quê" do COCREATE em "exatamente como" com arquivos específicos, ACs mensuráveis e tasks executáveis.

```
COCREATE (handoff): "Leia AgendarParcelasProcessor antes de criar tasks"
REFINE:             "Task 1: push job para queue com shape {parcelaId, vencimento, ...}
                     AC: job enfileirado sem duplicidade verificado por enviado_automaticamente"
```

---

## Dicas de uso

- **Responda as HMW** — elas moldam o plano. Não pule.
- **Questione a ordem dos loops** — loop 1 deve ser o de menor risco e maior valor
- **Especifique as telas** — quanto mais concreto no COCREATE, melhor o REFINE
- **Use "não" nos limites** — o que cada loop NÃO inclui é tão importante quanto o que inclui
