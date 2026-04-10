# OBSERVE — Fase de Exploração

**Obrigatória?** Não — mas recomendada antes de iniciar um projeto novo.

O OBSERVE é a fase do **porquê**. Antes de qualquer planejamento, é preciso entender o que existe hoje e onde se quer chegar. A diferença entre os dois é o que o projeto vai construir.

---

## Papel no Framework

| Fase | Quem faz | Quando | Output |
|------|----------|--------|--------|
| **OBSERVE** | AI + usuário | antes de decidir | entendimento da realidade — o que existe, regras de negócio, o que está fora do escopo |
| **COCREATE** | AI especialista + usuário | antes de construir | decisões estratégicas + esboços técnicos que as sustentam |
| **RESEARCH** | AI | antes do REFINE | guardrails: o que não reabrir, onde aprofundar + as perguntas já respondidas |

---

## O que o OBSERVE faz

1. **Discussão estruturada** — mapeia cenário atual e futuro via perguntas
2. **Salva OBSERVE.md** — arquivo único com visão + research de alto nível
3. **Executa research de negócio** — escaneia o produto no nível de menus, domínios e fluxos
4. **Handoff** — entrega o OBSERVE.md para o COCREATE ou REFINE usar como base

---

## Quando usar

- Antes de iniciar um projeto que não existe ainda no ORBTI
- Quando o problema não está claro — o OBSERVE força a clareza
- Quando há múltiplas direções possíveis e é preciso escolher uma
- **Não é necessário** para projetos já em andamento ou bugs (use `/orbti:refine-fix`)

---

## Como usar

```bash
/orbti:observe {tópico}

# Exemplos:
/orbti:observe Eficiência de débito
/orbti:observe Sistema de webhooks para parceiros
/orbti:observe Retorno DAVOS — integração CNAB
```

O OBSERVE abre uma conversa. Responda as perguntas — o sistema sintetiza e salva.

---

## O que o OBSERVE.md contém

O OBSERVE.md é um **arquivo único** que combina a visão do projeto com o research de alto nível executado automaticamente após a discussão:

```
Visão (produzida pela discussão):
  ├── Como é hoje — cenário atual, atores, problemas
  ├── Para onde vamos — o que muda, o que passa a existir
  ├── Goals — objetivos derivados da diferença
  ├── Marcos — sequência de entregas
  ├── Limites — o que o projeto NÃO deve fazer
  ├── Riscos — problemas ocultos identificados
  └── Solution Intent — WHO / WHAT / FEEL

Research de alto nível (executado automaticamente):
  ├── Domínios afetados — quais áreas de negócio são tocadas
  ├── Itens de navegação/menu que serão afetados
  ├── Fluxos existentes que mudarão
  └── Perguntas abertas — o que o research não conseguiu responder
```

---

## Como o OBSERVE alimenta o RESEARCH.md

O OBSERVE é a primeira camada do RESEARCH progressivo:

```
OBSERVE → pergunta "que domínios são afetados?"
        → escaneia navegação, módulos, fluxos existentes
        → adiciona ao RESEARCH.md: "## Nível de Negócio"
```

O REFINE (várias fases depois) já tem essa base. Não precisa redescobrir o que o OBSERVE mapeou.

---

## Para a próxima fase — ponteiros de onde buscar

O OBSERVE termina com uma seção `## Para a próxima fase` no OBSERVE.md. Ela contém **ponteiros explícitos** de onde o COCREATE (ou REFINE, se pular o cocreate) deve buscar na fase seguinte.

**Por que importa:**
O research progressivo só funciona se cada fase sabe *onde* pesquisar. Sem ponteiros, a fase seguinte explora o codebase aleatoriamente — o que anula o ganho de tokens do research progressivo.

**Formato:**
```markdown
## Para a próxima fase
- `apps/api-menos-juros/src/v2/carteira/debito/` — estrutura do sub-módulo (confirmar antes de propor novos arquivos)
- `supabase/migrations/` — última migration de fluxo_pagamentos (numeração da próxima)
- `apps/crm-menos-juros/client/src/domains/carteira/` — padrão de domain front existente
```

**Regra:** paths concretos, não tópicos. "Ver `apps/api-menos-juros/src/v2/carteira/debito/`" é útil. "Verificar a arquitetura do backend" não é.

A cadeia completa de ponteiros:

```
OBSERVE.md → ## Para a próxima fase
                    ↓
            COCREATE lê + aprofunda + deixa handoff estruturado por loop
                    ↓
            COCREATE.md → ## Handoff para Refine → Leia antes de criar tasks
                    ↓
            REFINE lê handoff → abre direto nos arquivos certos
                    ↓
            RESEARCH.md → ### Para a próxima fase (próximo loop do refine)
```

---

## Perguntas que o OBSERVE responde

| Pergunta | Onde vai |
|----------|----------|
| Como funciona hoje? | `## Como é hoje` |
| O que precisa mudar? | `## Para onde vamos` |
| Quais são os objetivos? | `## Goals` |
| O que este projeto NÃO deve fazer? | `## Limites` |
| Quais riscos ocultos existem? | `## Riscos` |
| Quem usa? O que faz? Como deve se sentir? | `## Solution Intent` |

---

## Próximos passos após o OBSERVE

```
OBSERVE completo
      │
      ├── /orbti:cocreate {slug}   ← quando a solução ainda não está clara
      │                               (como estratégico, antes do técnico)
      │
      └── /orbti:refine {slug}     ← quando a solução já está suficientemente
                                      clara para planejar o loop 1
```

O OBSERVE não bloqueia o REFINE. Se a solução já está clara após a discussão, vá direto para o REFINE.

---

## Estrutura de arquivos criada

```
.orbti/projects/{slug}/
└── OBSERVE.md    ← criado pelo observe (visão + research de negócio)
```

---

## Dicas de uso

- **Seja específico no tópico** — `"Retorno DAVOS"` é melhor que `"integração"`
- **Responda com exemplos concretos** — "hoje fazemos X manualmente" > "o processo é manual"
- **Não planeje na discussão** — o OBSERVE é para entender, não para decidir como construir
- **Diga os limites** — o que este projeto não deve fazer é tão importante quanto o que deve
