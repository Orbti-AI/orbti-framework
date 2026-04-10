# INTEGRATE — Fase de Fechamento do Loop

**Obrigatória?** Sim — presente em todo loop, sem exceção.

O INTEGRATE fecha o loop. Reconcilia o que foi planejado (REFINE) com o que foi construído (BUILD), executa code review estruturado, documenta o resultado e atualiza o estado do projeto.

**Importante:** o INTEGRATE não faz git commit ou push. Isso é responsabilidade de special flows separados.

---

## O que o INTEGRATE faz

1. **Verifica evidência de testes** — lê TEST-PLAN.md se existir
2. **Reconcilia** — compara cada AC do REFINE com o resultado real do BUILD
3. **Code review** — análise estruturada de qualidade, segurança e cobertura de ACs
4. **Cria INTEGRATE.md** — documentação do que foi construído
5. **Cria REVIEW.md** — resultado do code review
6. **Atualiza STATE.md** — marca o loop como completo

---

## Como usar

```bash
/orbti:integrate .orbti/projects/{slug}/01-loop1-F-REFINE.md
/orbti:integrate .orbti/projects/{slug}/02-loop1-B-REFINE.md
```

Cada specialist tem seu próprio INTEGRATE — rodar separadamente para F, B, T, A.

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
├── 01-loop1-F-INTEGRATE.md   ← reconciliação + resultado
└── 01-loop1-F-REVIEW.md      ← code review detalhado
```

**INTEGRATE.md contém:**
- O que foi construído (tabela: arquivo, propósito, linhas)
- ACs: cada uma passou ou falhou, com evidência
- Resultado do code review (link para REVIEW.md + status: PASS / PASS COM RESSALVAS / FAIL)
- Desvios em relação ao REFINE.md (com explicação)
- Deferred issues (minor findings para tratar depois)
- Próxima ação

---

## O que o INTEGRATE NÃO faz

| Ação | Por que não |
|------|------------|
| Git commit | Special flow separado — não é parte do loop padrão |
| Git push | Idem |
| Criar PR | Idem |

Para commitar após integrar, use os special flows configurados em `SPECIAL-FLOWS.md`.

---

## Reconciliação: Plano vs Real

Para cada AC do REFINE:

```
AC: "GET /remessas retorna 200 com lista paginada"
  Status: ✓ PASS
  Evidência: endpoint criado em carteira.controller.ts, teste manual confirmado

AC: "Filtro por status funciona"
  Status: ✓ PASS
  Evidência: implementado via query param ?status=

AC: "Estado de erro exibe toast"
  Status: ✗ FAIL
  Detalhe: implementado sem toast — exibe texto inline. Desvio documentado.
```

---

## STATE.md após o INTEGRATE

O loop é marcado como completo:

```
Loop 1 — Front:
  REFINE ✓  BUILD ✓  INTEGRATE ✓

Loop 1 — Back:
  REFINE ✓  BUILD ✓  INTEGRATE ✓

Loop 2 — Front:
  REFINE ○  BUILD ○  INTEGRATE ○  ← próximo
```

---

## Após o INTEGRATE

```
INTEGRATE completo
      │
      ├── Mais loops no projeto → /orbti:refine {slug}  (loop N+1)
      │
      └── Último loop → transição de fase (STATE.md atualizado, próximo projeto)
```
