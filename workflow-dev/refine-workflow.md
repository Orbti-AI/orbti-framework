# REFINE — Fase de Planejamento

**Obrigatória?** Sim — presente em todo loop, sem exceção.

O REFINE é o coração do framework. É onde o trabalho é planejado com precisão suficiente para ser executado pelo BUILD. Um bom REFINE elimina ambiguidade — o BUILD não toma decisões de design ou arquitetura, apenas executa.

---

## O que o REFINE faz

1. **Aprofunda o research técnico** — pesquisa arquivos, componentes, hooks, schemas existentes
2. **Detecta specialists** — determina quais refines criar (F, B, T, A) baseado no contexto
3. **Aplica filtros** — parâmetros `-F`, `-B`, `-T`, `--T` controlam quais refines são criados
4. **Cria os arquivos REFINE.md** — um por specialist, com ACs, tasks e limites claros
5. **Atualiza RESEARCH.md** — adiciona a camada técnica do loop atual

---

## Como usar

```bash
/orbti:refine {slug}           # todos os specialists do loop (exceto T se não configurado)
/orbti:refine {slug} -F        # apenas Front
/orbti:refine {slug} -B        # apenas Back
/orbti:refine {slug} -F -B     # Front e Back (sem Test, sem Agent)
/orbti:refine {slug} -T        # forçar criação do refine de Test
/orbti:refine {slug} --T       # excluir Test (mesmo se config habilitada)
/orbti:refine {slug} -A        # incluir Agent
```

---

## Naming Convention dos Arquivos

```
LL_SS-S-REFINE.md
```

| Parte | Significado | Valores |
|-------|------------|---------|
| `LL` | Número do loop | 01, 02, 03... |
| `SS` | Sequencial dentro do loop (evita colisão de nomes) | 01, 02, 03... |
| `S` | Specialist | F, B, T, A |

**Exemplos:**
```
01_01-B-REFINE.md   ← loop 1, seq 1, backend
02_01-B-REFINE.md   ← loop 2, seq 1, backend  ┐ mesmo loop,
02_02-F-REFINE.md   ← loop 2, seq 2, frontend ┘ rodam em paralelo
03_01-T-REFINE.md   ← loop 3, seq 1, testes (só se habilitado)
03_01-A-REFINE.md   ← loop 3, seq 1, agent
```

---

## Specialists

| Specialist | Código | O que cobre |
|------------|--------|-------------|
| Frontend | `F` | Telas, componentes React, navegação, estado |
| Backend | `B` | Endpoints, services, migrations, lógica de negócio |
| Test | `T` | Testes automatizados — criados apenas se configurado |
| Agent | `A` | Workers, automações, integrações assíncronas |

**Regra do Test:** o refine T só é criado automaticamente se `test: enabled: true` no `config.md`. Para forçar manualmente, use `-T`. Para excluir, use `--T`.

---

## HMW Técnico — posição condicional ao COCREATE.md

O REFINE abre perguntas HMW técnicas ("como implementar?") em momento diferente dependendo de ter ou não cocreate.

### COM COCREATE.md — HMW upfront + HMW residual

O COCREATE.md já deixou decisões técnicas abertas documentadas (seção "Handoff para Refine"). Não faz sentido esperar o research para perguntar o que já se sabe que é uma decisão.

```
HMW iniciais (do COCREATE.md)
      ↓
Research técnico (anota novos unknowns, não interrompe)
      ↓
HMW residuais (se o research revelou novos unknowns)
      ↓
Criar tasks
```

### SEM COCREATE.md — HMW após research

Sem base estratégica, HMW upfront seria genérico. Research primeiro, perguntas depois.

```
Research técnico
      ↓
HMW técnico (baseado no que o research revelou)
      ↓
Criar tasks
```

### Escopo do HMW técnico

Perguntas que exigem conhecer o codebase — o COCREATE não responde:
- Trade-offs: "reutilizar X ou criar Y?"
- Idempotência, edge cases específicos do código
- Acoplamento entre módulos
- Schema: coluna nova vs campo JSONB
- Camada correta: domain service vs use case vs infra

**Pular silenciosamente** se não há decisão não óbvia.

---

## Como o REFINE aprofunda o RESEARCH

Antes de criar os arquivos REFINE.md, o sistema executa o research técnico do loop. O ponto de partida depende do que foi rodado antes:

### COM COCREATE.md — research focado (mínimo de tokens)

```
REFINE lê: COCREATE.md → seção "Handoff para Refine" → "Leia antes de criar tasks"
         → abre direto nos 3-5 arquivos listados
         → NÃO re-escaneia o que o COCREATE já descobriu
         → adiciona ao RESEARCH.md: "## Refine Loop N"
```

### SEM COCREATE, COM OBSERVE.md — research médio

```
REFINE lê: OBSERVE.md → seção "Para a próxima fase"
         → usa os ponteiros como ponto de entrada do research
         → explora apenas os paths indicados + dependências imediatas
         → adiciona ao RESEARCH.md: "## Refine Loop N"
```

### SEM OBSERVE E SEM COCREATE — research completo (standalone)

```
REFINE faz: pesquisa completa em uma passagem — negócio + estratégico + técnico
          → não há handoff para ler, não há base para reutilizar
          → cria RESEARCH.md do zero
          → adiciona "## Para a próxima fase" para o próximo loop do refine
```

**Custo de tokens por modo:**
- Com COCREATE: mínimo — lê só os 3-5 arquivos do handoff
- Com OBSERVE: médio — lê ponteiros + explora paths indicados
- Standalone: máximo — pesquisa completa em uma passagem

---

## Para a próxima fase — ponteiros entre loops do REFINE

Ao final de cada seção do RESEARCH.md, o refine adiciona `### Para a próxima fase`:

```markdown
### Para a próxima fase
- `apps/api-menos-juros/src/v2/carteira/debito/efetividade/` — confirmar se existe antes de propor novos endpoints no Loop 2
- `supabase/migrations/` — verificar que a migration do Loop 1 foi aplicada antes de criar queries de efetividade
```

O refine do Loop 2 lê isso primeiro — não re-descobre o que o Loop 1 já resolveu.

Esse research não é anunciado ao usuário — acontece silenciosamente antes de criar os refines.

---

## O que um REFINE.md contém

```yaml
---
project: eficiencia-debito
refine: "01"
specialist: F
loop: 1
type: execute
depends_on: []
autonomous: false
---
```

**Seções principais:**
- `<objective>` — goal, propósito e output esperado
- `<context>` — arquivos relevantes para o BUILD ler
- `<skills>` — skills obrigatórias (ex: DDD para API, design para frontend)
- `## Component Map` — (só F) mapa de componentes existentes vs a criar
- `## Acceptance Criteria` — critérios mensuráveis e verificáveis
- `## Tasks` — lista executável de o que fazer
- `## Boundaries` — o que este refine NÃO deve fazer

---

## Acceptance Criteria — Como escrever

Um AC deve ser verificável pelo BUILD sem ambiguidade:

```
✗ Ruim:  "A tela deve exibir os dados corretamente"
✓ Bom:   "GET /remessas retorna 200 com array de remessas paginado, max 20 por página"

✗ Ruim:  "O filtro deve funcionar"
✓ Bom:   "Filtro por status=pendente exibe apenas remessas com status 'pendente'"

✗ Ruim:  "Deve ter tratamento de erro"
✓ Bom:   "Quando API retorna 500, exibe toast 'Erro ao carregar remessas' e mantém dados anteriores"
```

---

## Frontend: Component Map

Para refines F, o REFINE mapeia componentes existentes antes de criar tasks:

| Elemento | Status | Componente | Import |
|----------|--------|------------|--------|
| Tabela | reutilizar | `<Table>` + TanStack | `@/components/ui/table` |
| Badge status | reutilizar | `<Badge variant="...">` | `@/components/ui/badge` |
| Sheet de detalhes | reutilizar | `<Sheet>` | `@/components/ui/sheet` |
| Gráfico de efetividade | criar | `<EfetividadeChart>` | — |

**Regra:** reutilizar existentes é sempre preferível. Criar novos só quando nenhum existente serve.

---

## Dependências entre Specialists

Quando F e B estão no mesmo loop e o F depende de tipos gerados pelo B:

```yaml
# No frontmatter do refine F:
integrates_with: "02"   # NN do refine B
```

O BUILD usa isso para saber que o F deve aguardar o B entregar os tipos antes de fazer a integração real (até lá, usa mocks).

---

## Fonte de contexto para o REFINE

O REFINE lê, nessa ordem, para construir os refines:
1. `OBSERVE.md` — visão e cenário do projeto
2. `COCREATE.md` — plano estratégico (se existir)
3. `RESEARCH.md` — pesquisas anteriores (negócio + estratégico)
4. Integrações anteriores — `NN-INTEGRATE.md` do loop anterior (se relevante)

---

## Após o REFINE

```
REFINE criado
      │
      ▼
Usuário revisa e aprova os arquivos REFINE.md
      │
      ▼
/orbti:build {slug}       ← executa o próximo loop pendente
/orbti:build {refine}     ← executa refine específico
```

**O BUILD não começa sem aprovação explícita do usuário.**
