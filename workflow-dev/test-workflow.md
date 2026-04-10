# TEST — Fase de Validação

**Obrigatória?** Não — mas recomendada quando há refine T no loop ou quando o trabalho tem impacto crítico.

O TEST executa e valida o que foi construído pelo BUILD. Combina checkpoint manual com execução automatizada via Playwright (quando configurado).

---

## O que o TEST faz

1. **Carrega cenários** — do refine T do loop ou gera cenários básicos a partir do REFINE.md
2. **Checkpoint manual** — exibe checklist para o usuário validar manualmente
3. **Playwright** — se configurado, oferece execução automatizada dos testes E2E
4. **Cria TEST-PLAN.md** — documenta o que foi testado e o resultado

---

## Quando usar

- Após o BUILD, antes do INTEGRATE — para validar antes de fechar o loop
- Quando há refine T no loop (testes foram construídos pelo BUILD)
- Quando o trabalho tem impacto em fluxos críticos (pagamentos, conciliação, etc.)
- **Não bloqueia o INTEGRATE por padrão** — mas o INTEGRATE avisa se não há TEST-PLAN

---

## Como usar

```bash
/orbti:test {slug}                               # testa o loop mais recente
/orbti:test .orbti/projects/{slug}/NN-loopN-T-REFINE.md  # testa refine específico
```

---

## Checkpoint Manual

O TEST sempre exibe um checklist dos cenários a validar:

```
════════════════════════════════════════
TEST CHECKPOINT — {slug} Loop N
════════════════════════════════════════

[ ] Cenário 1: Listagem de remessas carrega corretamente
    Esperado: tabela exibe remessas com status, data e valor

[ ] Cenário 2: Filtro por status=pendente funciona
    Esperado: apenas remessas pendentes aparecem

[ ] Cenário 3: Estado vazio exibe mensagem correta
    Esperado: "Nenhuma remessa pendente" com CTA

────────────────────────────────────────
Confirme cada cenário. Digite:
  "passou" — todos passaram
  "falhou [N]" — cenário N falhou
  "parcial [N,M]" — cenários N e M falharam
════════════════════════════════════════
```

---

## Playwright (E2E Automatizado)

Disponível quando:
- `e2e: enabled: true` no `config.md`
- `playwright-cli` registrado no `SPECIAL-FLOWS.md`

```
Quer executar os testes E2E automaticamente?
[1] Sim — Playwright executa os cenários agora
[2] Não — marcar resultado manualmente
```

---

## TEST-PLAN.md

Criado em `.orbti/projects/{slug}/NN-loopN-TEST-PLAN.md`:

```markdown
---
status: passed | partial | failed
tested: YYYY-MM-DD
---

| # | Cenário | Tipo | Status |
|---|---------|------|--------|
| 1 | Listagem carrega | manual | ✓ |
| 2 | Filtro funciona | e2e | ✓ |
| 3 | Estado vazio | manual | ✗ |

## Falhas
- Cenário 3: estado vazio exibe spinner em loop ao invés da mensagem
```

O INTEGRATE lê o TEST-PLAN.md e inclui o resultado no INTEGRATE.md.

---

## O que acontece sem TEST-PLAN

O INTEGRATE **não bloqueia** se não houver TEST-PLAN, mas avisa:

```
⚠ Nenhum TEST-PLAN encontrado para este refine.
  Prosseguindo sem evidência de testes — documente manualmente se necessário.
```

---

## Após o TEST

```
TEST completo
      │
      └── /orbti:integrate {refine}  ← obrigatório para fechar o loop
```
