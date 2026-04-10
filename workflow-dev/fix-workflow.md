# Fix Workflow (orbti fix)

Documenta o fluxo de correção de bugs via ORBTI, da descoberta até o commit.

---

## Fluxo Completo

```
Bug reportado pelo usuário
        │
        ▼
1. /orbti:refine-fix <descrição>
        │
        ├─ Explora o codebase (find root cause)
        ├─ Gera slug: "pago-grupo-nao-recebidas"
        └─ Cria .orbti/fix/{slug}/01-FIX.md
              frontmatter:
                worktree:
                  enabled: true
                  branch: main
                  repository: apps/crm-menos-juros
        │
        ▼
2. Usuário revisa FIX.md (opcional) → aprova
        │
        ▼
3. /orbti:build .orbti/fix/{slug}/01-FIX.md
        │
        ├─ Lê worktree.repository (ex: apps/crm-menos-juros)
        ├─ Cria git worktree isolado em .worktrees/{slug}/
        │    git worktree add .worktrees/{slug} -b fix/{slug}
        ├─ Configura worktree para rodar como o principal:
        │    ln -s {repo}/node_modules .worktrees/{slug}/node_modules
        │    cp {repo}/.env.local .worktrees/{slug}/.env.local
        ├─ Agente background aplica mudanças dentro de .worktrees/{slug}/
        └─ Oferece servidor de revisão: PORT=5001 npm run dev
        │
        ▼
4. /orbti:test .orbti/fix/{slug}/01-FIX.md
        │
        ├─ Escreve/roda testes contra os ACs
        └─ Resultado: passed / failed / manual UAT
        │
        ▼
5. /orbti:integrate .orbti/fix/{slug}/01-FIX.md
        │
        ├─ Cria INTEGRATE.md
        ├─ Merge branch → main no submodule
        ├─ Commit no submodule
        └─ Commit do ponteiro do submodule no monorepo + push
```

---

## Estrutura de Arquivos

```
.orbti/
└── fix/
    └── {slug}/
        ├── 01-FIX.md        ← criado pelo refine-fix
        └── 01-INTEGRATE.md  ← criado pelo integrate
```

---

## Branch no Submodule

Quando `worktree: enabled: true` e `repository: apps/crm-menos-juros`:

```
# O build deve criar:
cd apps/crm-menos-juros
git checkout -b fix/{slug}

# Aplicar mudanças no branch (não no main)

# O integrate depois faz:
git checkout main
git merge fix/{slug}
git push
# E no monorepo: commit do ponteiro atualizado
```

---

## Ciclo de vida do branch

```
BUILD      → cria fix/{slug} em {worktree.repository}
               └─ branch visível no VS Code Source Control
               └─ pergunta: "Quer subir servidor para revisar?"
                    ├─ Sim → sobe na porta alternativa (ex: 5001)
                    │        usuário revisa em http://localhost:5001
                    └─ Não → vai direto para TEST

TEST       → roda testes no branch ativo
               └─ branch ainda vivo

INTEGRATE  → merge fix/{slug} → main no submodule
               git push
               git branch -d fix/{slug}
               monorepo: commit ponteiro submodule + push
               └─ branch encerrado aqui
```

O branch só é encerrado no INTEGRATE — nunca antes.
O `orbti:build` cria, o `orbti:integrate` fecha.

---

## Exemplo Real

| Step | Comando | Resultado |
|------|---------|-----------|
| 1 | `/orbti:refine-fix no dash do fluxo...` | `.orbti/fix/pago-grupo-nao-recebidas/01-FIX.md` |
| 2 | Revisão + aprovação | FIX.md ajustado: 1 task, mudança cirúrgica |
| 3 | `/orbti:build 01-FIX.md` | Agente background, mudança em `detalhes.tsx` |
| 3b | `git checkout -b fix/pago-grupo-nao-recebidas` | Branch criado no submodule (manual workaround) |
| 4 | `/orbti:test 01-FIX.md` | — pendente |
| 5 | `/orbti:integrate 01-FIX.md` | — pendente |

---

## Status Recebidos (regra de negócio)

A classificação de parcelas em "Recebidas" vs "Não Recebidas" é por `status_pagamento`:

| Status | Grupo |
|--------|-------|
| `pago` | Recebidas |
| `baixa_manual` | Recebidas |
| `pago_antecipacao` | Recebidas |
| `baixa_quitacao` | Recebidas |
| `pago_parcial` | Recebidas |
| `agendado` | Não Recebidas |
| `nao_agendado` | Não Recebidas |
| `rejeitado` | Não Recebidas |
| `sem_saldo` | Não Recebidas |
| `retorno_debito` | Não Recebidas |
| `cancelado` | Não Recebidas |
| `bloqueado` | Não Recebidas |
