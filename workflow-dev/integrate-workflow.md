# INTEGRATE — Fase de Fechamento do Loop

**Obrigatória?** Sim — presente em todo loop, sem exceção.

O INTEGRATE fecha o loop inteiro. Reconcilia o que foi planejado (todos os REFINEs do loop) com o que foi construído (todos os BUILDs do loop), executa code review estruturado, documenta o resultado e atualiza o estado do projeto.

**Regra fundamental:** INTEGRATE é **um por loop** — não por specialist. Só executa quando TODOS os specialists do loop (F, B, T, A) encerraram o BUILD. Loop parcial = apenas pause.

**Importante:** o INTEGRATE não faz git commit ou push. Isso é responsabilidade de special flows separados.

---

## O que o INTEGRATE faz

1. **Verifica se o loop está completo** — todos os specialists concluíram BUILD
2. **Verifica evidência de testes** — lê TEST-PLAN.md se existir
3. **Reconcilia** — compara cada AC de todos os REFINEs do loop com o resultado real
4. **Code review** — análise estruturada de qualidade, segurança e cobertura de ACs
5. **Cria `loopN-INTEGRATE.md`** — um único documento de fechamento do loop
6. **Cria REVIEWs** — `NN-loopN-S-REVIEW.md` por specialist
7. **Atualiza STATE.md** — marca o loop como completo

---

## Como usar

```bash
/orbti:integrate {slug}          # fecha o loop atual completo
/orbti:integrate eficiencia-debito
```

Um único comando por loop — não por specialist.

---

## Loop parcial → apenas pause

Se algum specialist ainda está em BUILD quando o INTEGRATE é chamado:

```
════════════════════════════════════════
⏸ INTEGRATE INDISPONÍVEL — LOOP PARCIAL
════════════════════════════════════════
Loop 1 ainda não está completo.

Specialists concluídos:  F ✓  B ✓
Specialists pendentes:   T ○

INTEGRATE só executa quando todos os specialists fecharem BUILD.

[1] Pausar — salvar estado e continuar depois
[2] Cancelar
════════════════════════════════════════
```

**Pause** salva o progresso parcial no STATE.md sem criar INTEGRATE.md. Quando o specialist pendente terminar, o INTEGRATE executa normalmente.

---

## Code Review Estruturado

O INTEGRATE sempre executa code review antes de fechar o loop. A fonte é escolhida automaticamente:

```
1ª opção: /skill code-reviewer (se disponível nas skills do projeto)
2ª opção: superpowers:code-reviewer agent
3ª opção: review inline estruturado
```

**O review avalia 4 dimensões:**

| Dimensão | O que verifica |
|----------|---------------|
| Qualidade | Clareza, consistência com padrões do projeto, sem complexidade desnecessária |
| Segurança | Inputs validados, nada exposto, sem injeção, auth preservada |
| Cobertura | Cada AC do REFINE.md foi satisfeita? Tem AC faltando? |
| Escopo | Alguma mudança fora dos limites do REFINE.md? |

**Severidades:**

| Severidade | Consequência |
|------------|-------------|
| `blocker` | Loop não fecha. Abrir `/orbti:refine-fix` para corrigir |
| `major` | Incluído no INTEGRATE.md. Usuário decide se prossegue |
| `minor` | Registrado como deferred issue. Não bloqueia |

---

## Arquivos criados

```
.orbti/projects/{slug}/
├── loop1-INTEGRATE.md        ← fechamento do loop inteiro (único por loop)
├── 01-loop1-F-REVIEW.md      ← code review do Front
└── 02-loop1-B-REVIEW.md      ← code review do Back
```

**`loopN-INTEGRATE.md` contém:**
- Objetivo do loop (do COCREATE ou REFINE)
- Tabela de specialists: cada um com status PASS/FAIL
- O que foi construído (tabela consolidada: arquivo, specialist, propósito)
- ACs consolidadas: cada uma de todos os specialists (passou ou falhou, com evidência)
- Resultado dos code reviews (links para cada REVIEW.md)
- Desvios em relação ao REFINE.md (com explicação)
- Deferred issues (minor findings para tratar depois)
- Próximo loop

---

## O que o INTEGRATE NÃO faz

| Ação | Por que não |
|------|------------|
| Git commit | Special flow separado — não é parte do loop padrão |
| Git push | Idem |
| Criar PR | Idem |

Para commitar após integrar, use os special flows configurados em `SPECIAL-FLOWS.md`.

---

## Reconciliação: Plano vs Real (por specialist)

Para cada AC de cada REFINE do loop:

```
Specialist B — AC 1: "POST /automacao enfileira job corretamente"
  Status: ✓ PASS
  Evidência: unit test passando, job shape confirmado

Specialist F — AC 2: "Tela exibe status da automação em tempo real"
  Status: ✗ FAIL
  Detalhe: polling implementado mas sem feedback visual de loading. Desvio documentado.
```

---

## STATE.md após o INTEGRATE

O loop inteiro é marcado como completo (não por specialist):

```
Loop 1 — Automação D-5:
  F: REFINE ✓  BUILD ✓
  B: REFINE ✓  BUILD ✓
  INTEGRATE ✓  ← loop fechado

Loop 2 — Efetividade:
  F: REFINE ○  BUILD ○
  B: REFINE ○  BUILD ○
  INTEGRATE ○  ← próximo
```

---

## Após o INTEGRATE

```
INTEGRATE do Loop N completo
      │
      ├── Mais loops no projeto → /orbti:refine {slug}  (loop N+1)
      │
      └── Último loop → transição de fase (STATE.md atualizado, próximo projeto)
```
