# PROGRESS — Status por Projeto e Loop

Exibe o estado atual de todos os projetos do ORBTI, organizado por projeto e por loop. Sempre termina com UMA sugestão de próxima ação.

---

## Como usar

```bash
/orbti:progress              # todos os projetos + próxima ação do projeto ativo
/orbti:progress {slug}       # detalhes completos de um projeto específico
/orbti:progress --all        # todos os projetos com detalhes de loops
```

---

## Output: visão geral

```
════════════════════════════════════════
ORBTI PROGRESS
════════════════════════════════════════

┌────────────────────────────────────────────────────────────────┐
│  #   Projeto               Loops   Status           Pos        │
│  11  eficiencia-debito      3/4+    🔵 In Progress   ✓✓○       │
│  10  baixa-antecipacao      1/1     ⏸ Paused         ✓✓○       │
│  12  sistema-webhooks       0/2     📋 Planned        ○○○       │
└────────────────────────────────────────────────────────────────┘

Ativo: eficiencia-debito

  loop 1: REFINE ✓  BUILD ✓  INTEGRATE ✓  [dashboard remessas]
  loop 2: REFINE ✓  BUILD ✓  INTEGRATE ✓  [automação + FIDEM]
  loop 3: REFINE ✓  BUILD ○  INTEGRATE ○  [alertas-debito] ← aqui

────────────────────────────────────────
▶ NEXT: /orbti:build .orbti/projects/eficiencia-debito/04-loop3-F-REFINE.md
════════════════════════════════════════
```

---

## Output: projeto específico

```
════════════════════════════════════════
PROGRESS — eficiencia-debito
════════════════════════════════════════

Status: 🔵 In Progress  |  3 loops em andamento

  loop 1 — dashboard remessas
    01-loop1-F: REFINE ✓  BUILD ✓  INTEGRATE ✓
    02-loop1-B: REFINE ✓  BUILD ✓  INTEGRATE ✓

  loop 2 — automação + FIDEM
    03-loop2-F: REFINE ✓  BUILD ✓  INTEGRATE ✓
    04-loop2-B: REFINE ✓  BUILD ✓  INTEGRATE ✓

  loop 3 — alertas-debito (domínio real)
    05-loop3-F: REFINE ✓  BUILD ○  INTEGRATE ○  ← pendente

────────────────────────────────────────
▶ NEXT: /orbti:build .orbti/projects/eficiencia-debito/05-loop3-F-REFINE.md
════════════════════════════════════════
```

---

## Markers de status

| Marker | Significado |
|--------|------------|
| `✓` | Fase concluída |
| `○` | Fase pendente |
| `►` | Fase em execução |
| `✗` | Fase com falha |

| Ícone | Status do projeto |
|-------|------------------|
| 🔵 | In Progress — sendo trabalhado |
| ⏸ | Paused — pausado, retomar com `/orbti:resume` |
| 📋 | Planned — ainda não iniciado |
| ✅ | Complete — todos os loops integrados |
| 🔴 | Blocked — bloqueado por dependência ou issue |

---

## Regra: UMA próxima ação

O progress sempre termina com exatamente UMA sugestão — não uma lista de opções.

A sugestão é determinada pelo loop position do projeto ativo:

| Estado | Sugestão |
|--------|----------|
| REFINE ✓, BUILD ○ | `/orbti:build {refine}` |
| BUILD ✓, INTEGRATE ○ | `/orbti:integrate {refine}` |
| Loop N completo | `/orbti:refine {slug}` |
| Sem observe | `/orbti:observe {slug}` |
| Bloqueado | "Resolve o bloqueio: {issue}" |
