# ORBTI Framework

Framework de desenvolvimento assistido por IA estruturado em loops progressivos.

---

## O Loop Central (sempre presente)

```
REFINE ──▶ BUILD ──▶ INTEGRATE
```

Todo trabalho passa por essas três fases. Sem exceção. Um "loop" é uma execução completa das três.

---

## Fluxo Completo (com fases opcionais)

```
[OBSERVE] ──▶ [COCREATE] ──▶ REFINE ──▶ BUILD ──▶ [TEST] ──▶ INTEGRATE
   │               │            │           │           │           │
 porquê        como         planeja     executa     valida      fecha
 cenário      estratégico   (obrig.)    (obrig.)   (opcion.)   (obrig.)
```

| Fase | Obrigatória | Propósito |
|------|-------------|-----------|
| OBSERVE | Não | Mapeia cenário atual e futuro |
| COCREATE | Não | Materializa o plano por loop e specialist |
| **REFINE** | **Sim** | Planeja o que será construído |
| **BUILD** | **Sim** | Executa o plano |
| TEST | Não | Valida o que foi construído |
| **INTEGRATE** | **Sim** | Fecha o loop, code review, documenta |

---

## Papéis das Fases Opcionais

Cada fase pré-REFINE tem um papel distinto e um output específico:

| Fase | Quem faz | Quando | Output |
|------|----------|--------|--------|
| **OBSERVE** | AI + usuário | antes de decidir | entendimento da realidade — o que existe, regras de negócio, o que está fora do escopo |
| **COCREATE** | AI especialista + usuário | antes de construir | decisões estratégicas + esboços técnicos que as sustentam |
| **RESEARCH** | AI | antes do REFINE | guardrails: o que não reabrir, onde aprofundar + as perguntas já respondidas |

## Como o entendimento se aprofunda

O RESEARCH.md é preenchido progressivamente — cada fase adiciona uma camada mais profunda:

```
OBSERVE   → enquadramento de negócio  (domínios, menus, fluxos, regras)
COCREATE  → decisões estratégicas     (o que construir, esboços técnicos)
REFINE    → contexto técnico acumulado (confirmações de cada loop)
```

O REFINE lê o RESEARCH.md antes de qualquer pesquisa técnica — evita re-descobrir o que as fases anteriores já responderam.

---

## Specialists e Naming

Cada loop tem um ou mais specialists rodando em paralelo:

| Código | Specialist | O que cobre |
|--------|------------|-------------|
| `F` | Frontend | Telas, componentes, navegação |
| `B` | Backend | APIs, migrations, regras de negócio |
| `T` | Test | Testes automatizados (config: `test: enabled: true`) |
| `A` | Agent | Workers, automações, integrações |

**Naming dos arquivos:**
```
LL_SS-S-REFINE.md

01_01-B-REFINE.md   ← loop 1, seq 1, back
02_01-B-REFINE.md   ← loop 2, seq 1, back  ┐ paralelo
02_02-F-REFINE.md   ← loop 2, seq 2, front ┘
03_01-T-REFINE.md   ← loop 3, seq 1, testes (se habilitado)
```

---

## Comandos

```bash
# Loop principal
/orbti:refine {slug}                    # planeja o próximo loop
/orbti:build {slug}                     # executa próximo loop pendente
/orbti:build {refine-path}              # executa refine específico
/orbti:integrate {refine-path}          # fecha o loop

# Enriquecimento
/orbti:observe {tópico}                 # mapeia cenário antes de iniciar
/orbti:cocreate {slug}                  # materializa plano estratégico
/orbti:test {slug}                      # executa testes do loop
/orbti:research {pergunta}              # research pontual

# Filtros do refine
/orbti:refine {slug} -F -B              # só front e back
/orbti:refine {slug} -T                 # forçar test refine
/orbti:refine {slug} --T                # excluir test refine

# Gestão de sessão
/orbti:pause                            # salva estado e encerra
/orbti:resume {slug}                    # retoma com contexto completo
/orbti:progress                         # status por projeto e loop
```

---

## Estrutura de arquivos por projeto

```
.orbti/projects/{slug}/
├── OBSERVE.md              ← visão + research de negócio
├── COCREATE.md             ← plano estratégico por loop/specialist
├── RESEARCH.md             ← research acumulado (todas as fases)
│
├── 01_01-B-REFINE.md       ─┐
├── 01_01-B-INTEGRATE.md    ├─ loop 1, back
├── 01_01-B-REVIEW.md      ─┘
│
├── 02_01-B-REFINE.md       ─┐
├── 02_01-B-INTEGRATE.md    ├─ loop 2, back  ┐ paralelo
├── 02_01-B-REVIEW.md      ─┘               │
├── 02_02-F-REFINE.md       ─┐              │
├── 02_02-F-INTEGRATE.md    ├─ loop 2, front ┘
├── 02_02-F-REVIEW.md      ─┘
│
└── HANDOFF-{data}.md       ← criado pelo pause
```

---

## Documentação por fase

| Arquivo | Fase | Obrigatória |
|---------|------|-------------|
| [observe-workflow.md](observe-workflow.md) | OBSERVE | Não |
| [cocreate-workflow.md](cocreate-workflow.md) | COCREATE | Não |
| [refine-workflow.md](refine-workflow.md) | REFINE | **Sim** |
| [build-workflow.md](build-workflow.md) | BUILD | **Sim** |
| [test-workflow.md](test-workflow.md) | TEST | Não |
| [integrate-workflow.md](integrate-workflow.md) | INTEGRATE | **Sim** |
| [research-workflow.md](research-workflow.md) | RESEARCH | Ferramenta transversal |
| [fix-workflow.md](fix-workflow.md) | FIX | Track alternativo (bugs) |
| [pause-workflow.md](pause-workflow.md) | PAUSE / RESUME / PROGRESS | Gestão de sessão |
