---
project: {slug}
total_loops: {N}
created: {date}
---

# LOOP MAP — {nome do projeto}

> Organização de construção em blocos (loops). Cada loop é um bloco autônomo.
> Uma invocação de `/orbti:build {projeto}` executa o próximo loop pendente — apenas ele.

---

## Loop 1 — {nome descritivo} {status}

| Refine | Tipo | Tasks | depends_on | worktree | Status |
|--------|------|-------|------------|---------|--------|
| 01-REFINE-FRONT.md | front | 4 | — | — | ✅ INTEGRATE |
| 01-REFINE.md | execute | 3 | — | — | ✅ INTEGRATE |

**Objetivo:** {o que este bloco entrega}
**Resultado:** {o que ficou pronto ao final}

---

## Loop 2 — {nome descritivo} {status}

| Refine | Tipo | Tasks | depends_on | worktree | Status |
|--------|------|-------|------------|---------|--------|
| 02-REFINE-FRONT.md | front | 2 | — | — | ⏳ BUILD |
| 02-REFINE.md | execute | 5 | 02-REFINE-FRONT | — | ⏳ BUILD |

**Objetivo:** {o que este bloco entrega}
**Resultado:** —

---

## Loop 3 — {nome descritivo} {status}

| Refine | Tipo | Tasks | depends_on | worktree | Status |
|--------|------|-------|------------|---------|--------|
| 03-REFINE.md | execute | 6 | — | — | 🔲 pendente |

**Objetivo:** {o que este bloco entrega}
**Resultado:** —

---

## Legenda de status

| Símbolo | Significado |
|---------|-------------|
| 🔲 | Pendente — refine não iniciado |
| ⚙ | REFINE criado — aguardando BUILD |
| ⏳ | BUILD executado — aguardando INTEGRATE |
| ✅ | INTEGRATE concluído — loop fechado |

## Progresso

```
Loop 1: [████████████] ✅  {N} tasks — integrado
Loop 2: [████████░░░░] ⏳  {N} tasks — built, aguardando integrate  
Loop 3: [░░░░░░░░░░░░] 🔲  {N} tasks — pendente
```

Total: {X} de {Y} loops completos ({Z}%)
