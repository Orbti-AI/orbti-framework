# BUILD — Fase de Execução

**Obrigatória?** Sim — presente em todo loop, sem exceção.

O BUILD executa o que foi planejado no REFINE. Não toma decisões de design ou arquitetura — segue o REFINE.md. Cada specialist do loop roda como um agente paralelo.

---

## O que o BUILD faz

1. **Detecta o modo** — projeto (próxima loop pendente) ou refine específico
2. **Verifica aprovação** — o REFINE.md deve ter sido aprovado pelo usuário
3. **Executa em paralelo** — um agente por specialist (F, B, A simultâneos)
4. **Constrói testes** — somente se há refine T no loop
5. **Registra resultados** — para o INTEGRATE usar na reconciliação

---

## Como usar

```bash
# Executar próximo loop pendente do projeto
/orbti:build {slug}
/orbti:build .orbti/projects/{slug}

# Executar um refine específico (ignora os demais)
/orbti:build .orbti/projects/{slug}/01-loop1-F-REFINE.md
```

**A diferença importa:**

| Argumento | Comportamento |
|-----------|--------------|
| Projeto / diretório | Identifica próximo loop sem INTEGRATE → executa todos os refines daquele loop |
| Arquivo REFINE.md | Executa apenas aquele refine — os outros do loop são ignorados |

---

## Execução Paralela

Dentro de um loop, F, B e A rodam em paralelo quando `parallel_build: enabled: true` no config:

```
Loop 1:
  ┌────────────────────────────────────────────┐
  │  Agente F: 01-loop1-F-REFINE.md            │
  │  Agente B: 02-loop1-B-REFINE.md  (paralelo)│
  │  Agente A: 03-loop1-A-REFINE.md  (paralelo)│
  └────────────────────────────────────────────┘
  Agente T: 04-loop1-T-REFINE.md (se existe, junto ou após)
```

Se paralelo não disponível → executa sequencialmente: F → B → A → T.

---

## Testes Automatizados (Condicional)

O BUILD **só constrói testes automatizados** se houver um refine T no loop:

```bash
# O BUILD verifica silenciosamente:
ls .orbti/projects/{slug}/*-loopN-T-REFINE.md
```

- **Existe refine T** → constrói os testes definidos nele
- **Não existe** → nenhum teste é construído (sem aviso, sem erro)

Isso é diferente de TDD: em TDD, os testes são escritos *antes* do BUILD. No fluxo padrão, o refine T define *o que* construir e o BUILD *constrói* os testes.

---

## Checkpoints

Quando o REFINE.md tem `autonomous: false`, o BUILD para em pontos específicos e aguarda confirmação:

```
════════════════════════════════════════
CHECKPOINT — {descrição}
════════════════════════════════════════
[detalhe do que foi feito / o que precisa de decisão]

Continuar? [sim / não / ajustar]
════════════════════════════════════════
```

---

## Regras do BUILD

| Regra | Detalhe |
|-------|---------|
| Sem aprovação, sem build | O usuário deve ter confirmado o REFINE antes |
| Um loop por execução | Loop N+1 requer novo `/orbti:build` |
| Segue o REFINE | Não inventa solução — executa o que foi planejado |
| Testes condicionais | Só constrói testes se refine T existir |
| Regista desvios | Se precisar desviar do REFINE, registra no napkin.md |

---

## Após o BUILD

```
BUILD completo
      │
      ├── /orbti:test {slug}         ← se há testes para executar
      │
      └── /orbti:integrate {refine}  ← fechar o loop
```

O INTEGRATE é obrigatório após o BUILD. Não há loop fechado sem INTEGRATE.
