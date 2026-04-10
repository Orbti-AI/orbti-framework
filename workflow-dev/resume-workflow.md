# RESUME — Retomada de Sessão

Documentação detalhada do fluxo de retomada. Para uso básico, ver [pause-workflow.md](pause-workflow.md).

---

## Fluxo completo

```
Nova sessão iniciada
        │
        ▼
/orbti:resume {slug?}
        │
        ├── Verifica .orbti/ existe
        ├── Lê STATE.md (fonte de verdade)
        ├── Detecta HANDOFF.md mais recente
        └── Compara datas → decide qual usar
        │
        ▼
Reconciliação: handoff vs STATE.md
        │
        ├── STATE.md mais recente → descarta handoff, avisa
        └── Handoff válido → usa como contexto adicional
        │
        ▼
Se múltiplos projetos ativos → exibe lista e pede seleção
        │
        ▼
Determina UMA próxima ação (não lista)
        │
        ▼
Exibe resumo + próxima ação
        │
        ▼
Arquiva handoff após prosseguir
```

---

## Output do RESUME

```
════════════════════════════════════════
ORBTI RESUMED — eficiencia-debito
════════════════════════════════════════

Projects Overview:
┌────────────────────────────────────────────────────────────────┐
│  #   Projeto                   Loops   Status           Pos    │
│  11  eficiencia-debito          3/4+    🔵 In Progress   ✓✓○   │
│  10  baixa-antecipacao          1/1     ⏸ Paused         ✓✓○   │
└────────────────────────────────────────────────────────────────┘

Ativo: eficiencia-debito
Parado em: loop 3 — refine F aprovado, build pendente

Loop status:
  loop 1: REFINE ✓  BUILD ✓  INTEGRATE ✓  [dashboard remessas]
  loop 2: REFINE ✓  BUILD ✓  INTEGRATE ✓  [automação + FIDEM]
  loop 3: REFINE ✓  BUILD ○  INTEGRATE ○  [alertas-debito] ← aqui

────────────────────────────────────────
▶ NEXT: /orbti:build .orbti/projects/eficiencia-debito/04-loop3-F-REFINE.md
  Build do front do loop 3 (alertas-debito)
────────────────────────────────────────
```

---

## Roteamento por loop position

| Estado do loop | Próxima ação |
|----------------|-------------|
| OBSERVE concluído, sem refine | `/orbti:cocreate` ou `/orbti:refine {slug}` |
| REFINE ✓, BUILD ○ | `/orbti:build {refine-path}` |
| BUILD ✓, TEST pendente | `/orbti:test {slug}` (se há refine T) |
| BUILD ✓, INTEGRATE ○ | `/orbti:integrate {refine-path}` |
| Loop N completo | `/orbti:refine {slug}` (loop N+1) |
| Projeto completo | Próximo projeto no ROADMAP |

---

## Múltiplos projetos ativos

Se STATE.md mostra mais de um projeto `🔵 In Progress` sem argumento:

```
Quais projetos você quer retomar?

  [1] eficiencia-debito  (loop 3, refine F build pendente)
  [2] baixa-antecipacao  (loop 1, integrate pendente)
```

Após seleção → foca no projeto escolhido e determina UMA próxima ação.

---

## Arquivos de referência

```
.orbti/
├── STATE.md                     ← fonte de verdade (lida primeiro)
└── projects/
    └── {slug}/
        ├── HANDOFF-{data}.md    ← contexto adicional (se válido)
        └── NN-loopN-S-REFINE.md ← para determinar a próxima ação exata
```
