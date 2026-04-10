# PAUSE / RESUME / PROGRESS — Gestão de Sessão

Ferramentas para pausar trabalho, retomar em outra sessão e ver o status do projeto.

---

## PAUSE — Pausar a Sessão

Quando precisar encerrar ou mudar de contexto, o PAUSE salva o estado completo para retomada futura.

```bash
/orbti:pause
/orbti:pause {slug}   # pausar projeto específico
```

**O que o PAUSE faz:**
1. Detecta projeto(s) ativo(s)
2. Coleta contexto da sessão (o que foi feito, o que está em progresso)
3. Cria `HANDOFF-{data}.md` na pasta do projeto
4. Atualiza STATE.md com status ⏸ Paused
5. Captura aprendizados no napkin.md
6. Oferece commit WIP (opcional)

**Estrutura do HANDOFF-{data}.md:**
```markdown
# ORBTI Handoff

## READ THIS FIRST
Projeto e core value — contexto para um Claude sem memória prévia

## Current State
Loop position atual por loop

## What Was Done
Lista do que foi concluído nesta sessão

## What's In Progress
O que está parcialmente feito

## What's Next
Próxima ação imediata + o que vem depois

## Key Files
Tabela de arquivos relevantes com propósito

## Resume Instructions
Passos exatos para retomar
```

---

## RESUME — Retomar a Sessão

```bash
/orbti:resume
/orbti:resume {slug}   # retomar projeto específico
```

**O que o RESUME faz:**
1. Lê STATE.md — fonte de verdade do estado
2. Detecta HANDOFF.md — usa se for mais recente que STATE.md
3. Exibe Projects Overview com status por loop
4. Determina **UMA** próxima ação — não uma lista
5. Arquiva o HANDOFF após prosseguir

**Roteamento por loop position:**

| Estado | Próxima ação |
|--------|-------------|
| REFINE ✓, BUILD ○ | `/orbti:build {refine}` |
| BUILD ✓, INTEGRATE ○ | `/orbti:integrate {refine}` |
| Loop N completo | `/orbti:refine {slug}` (loop N+1) |
| Sem refine | `/orbti:cocreate {slug}` ou `/orbti:refine {slug}` |

---

## PROGRESS — Status do Projeto

```bash
/orbti:progress              # todos os projetos + próxima ação
/orbti:progress {slug}       # detalhes de um projeto por loop
```

**Output padrão:**
```
════════════════════════════════════════
ORBTI PROGRESS
════════════════════════════════════════

┌────────────────────────────────────────────────────────────┐
│  #   Projeto               Loops   Status          Pos     │
│  11  eficiencia-debito      3/4+    🔵 In Progress  ✓✓○    │
│  10  baixa-antecipacao      1/1     ⏸ Paused        ✓✓○    │
└────────────────────────────────────────────────────────────┘

Ativo: eficiencia-debito

  loop 1: REFINE ✓  BUILD ✓  INTEGRATE ✓  [dashboard remessas]
  loop 2: REFINE ✓  BUILD ✓  INTEGRATE ✓  [automação + FIDEM]
  loop 3: REFINE ✓  BUILD ○  INTEGRATE ○  [alertas-debito] ← aqui

────────────────────────────────────────
▶ NEXT: /orbti:build .orbti/projects/eficiencia-debito/04-loop3-F-REFINE.md
════════════════════════════════════════
```

---

## STATUS Icons

| Ícone | Fase | Significado |
|-------|------|-------------|
| ✓ | Qualquer | Fase concluída |
| ○ | Qualquer | Fase pendente |
| ► | Qualquer | Fase em execução |
| ✗ | Qualquer | Fase com falha |
| 🔵 | Projeto | In Progress |
| ⏸ | Projeto | Paused |
| 📋 | Projeto | Planned (não iniciado) |
| ✅ | Projeto | Complete |

---

## Handoff: comportamento de defasagem

```
PAUSE  → cria HANDOFF-{data}.md
RESUME → compara data do handoff com STATE.md

  Se STATE.md mais recente → handoff DEFASADO → descarta, usa STATE.md
  Se handoff mais recente ou mesmo dia → usa handoff como contexto adicional
```

Quando o handoff é válido:
```
⚠ HANDOFF CONTEXT DETECTED — 2026-04-09
  Stopped at: eficiencia-debito — loop 3, refine F em progresso
```
