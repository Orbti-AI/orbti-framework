<div align="center">

# ORBTI

**Um loop de desenvolvimento estruturado para Claude Code.**

[![npm version](https://img.shields.io/npm/v/orbti-framework?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/orbti-framework)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/menosjuros/orbti-framework?style=for-the-badge&logo=github&color=181717)](https://github.com/menosjuros/orbti-framework)

</div>

---

## Instalação

**Pré-requisitos:** [Node.js](https://nodejs.org) 18+ e [Claude Code](https://claude.ai/code).

```bash
npx github:menosjuros/orbti-framework
```

Escolha **global** (`~/.claude/`) para usar em todos os projetos, ou **local** (`./.claude/`) apenas para o projeto atual.

```bash
# não-interativo
npx github:menosjuros/orbti-framework --global
npx github:menosjuros/orbti-framework --local
```

### Init

Reinicie o Claude Code após instalar, então rode:

```
/orbti:init
```

Isso cria `.orbti/` com `PROJECT.md`, `ROADMAP.md` e `STATE.md` via setup conversacional. Ao final você verá a estrutura:

```
.orbti/
├── PROJECT.md        # contexto e objetivos do projeto
├── ROADMAP.md        # milestones e projetos
├── STATE.md          # dashboard global — onde você está
├── config.md         # integrações opcionais
└── projects/         # uma pasta por projeto
```

Verifique com `/orbti:help` — você deve ver toda a referência de comandos.

### Config

Todas as integrações são **desabilitadas por padrão**. Habilite via:

```
/orbti:config
```

| Integração | O que faz | Padrão |
|---|---|---|
| **Agent Teams** | Pesquisa paralela no `/orbti:cocreate`, code review no `/orbti:integrate` | OFF |
| **Test Writer** | Escreve testes de integração durante `/orbti:build`, um por AC | OFF |
| **E2E (Playwright CLI)** | Testes browser via CLI externo do Playwright | OFF |

---

## O Fluxo Principal

Todo trabalho segue este ciclo. Sem atalhos.

```
REFINE ──▶ BUILD ──▶ TEST ──▶ INTEGRATE
Planejar    Executar   Verificar   Fechar
```

```
/orbti:refine     # define o que será construído — plano aprovado
/orbti:build      # executa o plano aprovado
/orbti:test       # valida contra os critérios de aceite
/orbti:integrate  # reconcilia e fecha o loop — nunca pule
```

Repita para cada unidade de trabalho.

---

## Pré-Refine — opcional

Antes de planejar, use esses comandos para clarificar o que quer, decidir como fazer, ou investigar dúvidas técnicas. Todos são opcionais — vá direto ao `/orbti:refine` quando o escopo já estiver claro.

```
observe → cocreate → research-phase → refine
  ↑           ↑            ↑
"o que       "qual        "como funciona
 quero"      caminho"      no código"
```

| Comando | Pergunta central | Quando usar |
|---|---|---|
| `/orbti:observe` | "O que quero alcançar?" | Escopo nebuloso, ideias vagas |
| `/orbti:cocreate` | "Qual opção/abordagem escolher?" | Decisão técnica com alternativas |
| `/orbti:research-phase` | "Como isso funciona aqui/lá?" | Investigar sem decidir ainda |
| `/orbti:refine` | "Como vamos fazer exatamente?" | Planejar a implementação |

**A diferença chave: `cocreate` vs `research-phase`**

`research-phase` → coleta informação, não decide:
> "Como o filtro de datas está implementado nas outras telas do CRM?"

`cocreate` → pesquisa para tomar uma decisão:
> "Devemos usar react-query ou SWR para essa feature?"
> → Compara opções → Produz recomendação

---

## Observe — *o quê*

**Quando usar:** antes de planejar, quando os objetivos precisam ser articulados.

```
/orbti:observe "nome-do-projeto"
```

Exploração conversacional guiada por você. O Claude faz perguntas para ajudar a articular metas, escopo e restrições. Você fala, o Claude escuta e organiza. Evita construir a coisa errada.

### Solution Intent — WHO / WHAT / FEEL

Toda solução com uma interface (visual, API, CLI, contrato de dados) passa por três perguntas antes de sintetizar o contexto:

```
WHO   — quem usa isso? (papel e contexto específico, não "os usuários")
WHAT  — o que precisam realizar? (o verbo principal — a ação que importa)
FEEL  — como a solução deve se comportar? (rápida e silenciosa? explícita? densa? forgiving?)
```

Essas respostas são salvas no `CONTEXT.md` como `Solution Intent` e usadas em todas as fases seguintes. O REFINE escreve ACs para esse WHO específico. O BUILD conecta cada decisão de implementação a esse contexto. Sem intent explícito, soluções convergem para o padrão mais genérico — não o que o problema exige.

**Não se aplica** a trabalho puramente técnico sem interface (refactor de arquitetura, hardening de segurança, infra).

**Output:** `CONTEXT.md` na pasta do projeto.

```
/orbti:observe-milestone   # mesma coisa, mas para o milestone inteiro
```

**Como fica após o observe:**

```
.orbti/projects/meu-projeto/
└── CONTEXT.md    # visão, restrições, escopo articulados
```

---

## Cocreate — *como*

**Quando usar:** quando há incógnitas técnicas que podem mudar o plano (seleção de biblioteca, padrão de arquitetura, abordagem de integração).

```
/orbti:cocreate "tema ou projeto"
```

O Claude implanta subagentes para pesquisar e comparar opções. Você decide. Diferente do `observe` (que é sobre objetivos), o `cocreate` é sobre decisões técnicas.

**Variações de profundidade** — detectadas automaticamente ou declaradas:
- **quick** — resposta direta, sem documento
- **standard** — pesquisa com `COCREATE.md`
- **deep** — múltiplos subagentes com verificação cruzada

**Output:** `COCREATE.md` com achados, recomendação e nível de confiança.

```
/orbti:assumptions   # superficia o que o Claude pretende fazer — pegue desalinhamentos antes do build
```

**Como fica o COCREATE.md:**

```markdown
# Cocreate: Auth Strategy

**Recommendation:** JWT with refresh rotation
**Confidence:** High

## Options Compared
| Option | Pros | Cons |
|--------|------|------|
| JWT + refresh | stateless, scalable | revocation complexity |
| Session-based | simple revocation | requires session store |

## Decision
...
```

---

## Research Phase — *investigar*

**Quando usar:** quando você sabe o que quer construir, mas tem dúvidas técnicas específicas para investigar antes de planejar.

```
/orbti:research-phase "tema ou pergunta"
```

Diferente do `cocreate` (que compara opções para tomar uma decisão), o `research-phase` coleta informação sem decidir. Use para entender como algo já está implementado no projeto, como uma API externa funciona, ou como um padrão específico funciona no codebase.

**Exemplos de uso:**
- "Como o filtro de datas está implementado nas outras telas do CRM?"
- "Como o sistema de autenticação atual lida com refresh tokens?"
- "Quais endpoints da API externa precisamos consumir?"

**Output:** `RESEARCH.md` com achados e contexto coletado — sem recomendação, sem decisão.

---

## Refine — *o plano*

**Quando usar:** após `observe` + `cocreate` (ou direto, se o escopo já está claro).

```
/orbti:refine
/orbti:refine "nome-do-projeto"   # especificar projeto
```

Cria um `REFINE.md` — o contrato do que será construído. Se você não consegue preencher todos os campos de uma tarefa, a tarefa está vaga demais para executar.

### Estrutura do REFINE.md

```markdown
---
project: auth
refine: 01
type: execute          # execute | tdd | research
wave: 1                # onda de execução (para paralelo)
depends_on: []         # IDs de refines que este depende
files_modified: []     # arquivos que este refine toca
autonomous: true       # false se tem checkpoints que exigem input
---

<objective>
## Goal    — o que este refine entrega
## Purpose — por que importa para o projeto
## Output  — artefatos criados/modificados
</objective>

<acceptance_criteria>
## AC-1: Login funciona
Given um usuário válido
When ele submete as credenciais
Then recebe um JWT token
</acceptance_criteria>

<tasks>
<task type="auto">
  <name>Criar endpoint de login com JWT</name>
  <files>src/api/auth/login.ts, src/lib/jwt.ts</files>
  <action>
    Criar POST /api/auth/login:
    - Aceitar { email, password }
    - Validar contra User model com bcrypt
    - Retornar JWT (15min) em sucesso
    - Retornar 401 em credenciais inválidas
  </action>
  <verify>curl -X POST /api/auth/login retorna token</verify>
  <done>AC-1 satisfeito</done>
</task>
</tasks>

<boundaries>
## DO NOT CHANGE
- database/migrations/*
- src/lib/auth.ts
</boundaries>
```

### Tipos de tarefa (checkpoints)

| Tipo | Uso |
|------|-----|
| `auto` | Execução totalmente autônoma |
| `checkpoint:decision` | Decisão de implementação — apresenta opções, espera seleção |
| `checkpoint:human-verify` | Verificação visual/funcional — pausa, instrui como verificar |
| `checkpoint:human-action` | Ação manual inevitável (deploy, serviço externo) |

Refines com `autonomous: false` têm checkpoints — não podem rodar em background.

### Macros e ROADMAP

O `ROADMAP.md` define a estrutura completa: milestones e projetos dentro de cada um. O refine lê o ROADMAP para saber qual projeto é o próximo.

```markdown
# Roadmap

## v1.0 — MVP

### 01. auth-service     (not started)
### 02. dashboard        (not started)
### 03. api-layer        (not started)
```

Projetos com `depends_on: []` e sem conflito de arquivos podem rodar em paralelo (mesmo wave). Use `wave` para declarar a ordem de execução quando há dependências reais.

Após criar o refine, o Claude oferece:
```
Continue to BUILD?
[1] Approved, run BUILD  [2] Questions first  [3] Pause here
```

---

## Build — *a execução*

**Quando usar:** após aprovação explícita do REFINE.md.

```
/orbti:build
/orbti:build .orbti/projects/auth/01-REFINE.md   # caminho específico
/orbti:build-bg                                   # background (requer autonomous: true)
```

Executa as tarefas em ordem, com verificação em cada etapa.

### Configuração de testes durante o build

Controlado via `/orbti:config`:

| Test Writer | Agent Teams | Comportamento no BUILD |
|---|---|---|
| OFF (padrão) | OFF | Só build — sem escrita de testes |
| OFF | ON | Só build — Agent Teams ativo em outras fases |
| ON | OFF | Sequencial — escreve teste após cada tarefa |
| ON | ON | **Paralelo** — builder + test-writer simultâneos |

Com `Agent Teams + Test Writer ON`, o BUILD spawna 2 agentes:
- **builder** — executa as tarefas em ordem
- **test-writer** — escreve testes de integração enquanto o builder termina cada tarefa

### Se o build não estiver ok

Quando uma verificação de tarefa falha, o Claude para imediatamente e oferece:

```
════════════════════════════════════════
Task 2 FAILED: Create login endpoint
Expected: curl returns 200 with token
Actual: 500 Internal Server Error
════════════════════════════════════════

[1] Retry — tentar novamente
[2] Skip — marcar como falha, continuar (cria desvio)
[3] Stop — parar, preparar para debug
```

Você pode interagir, corrigir o problema, e retomar. A decisão é registrada no `STATE.md`.

### Habilitando Agent Teams

```
/orbti:config → Agent Teams → Enable
```

Isso também escreve `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` em `.claude/settings.json`. Reinicie o Claude Code após habilitar.

Com Agent Teams ativo:
- **cocreate** — subagentes paralelos pesquisam opções
- **integrate** — code review roda junto com reconciliação
- **build (+ test writer)** — builder + test-writer em paralelo

Após o build, o Claude oferece:
```
Continue to TEST?
[1] Yes, run /orbti:test  [2] Pause here
```

---

## RUNBOOK — *aprendizado contínuo*

O ORBTI mantém um runbook vivo em `.orbti/RUNBOOK.md` que acumula aprendizado de sessão para sessão. Não é um log — é um guia de comportamento curado.

### Como funciona

| Fase | Comportamento |
|------|--------------|
| **OBSERVE** | Lê o RUNBOOK silenciosamente no início da sessão — absorve o aprendizado anterior sem anunciar |
| **BUILD** | Após cada task, se houve correção, desvio inesperado ou técnica que funcionou bem, loga no RUNBOOK |
| **INTEGRATE** | Curação: re-prioriza por impacto, merge de duplicatas, remove entradas obsoletas, limite de 10 por categoria |

### Estrutura do RUNBOOK.md

```markdown
## Execution & Validation
1. **[2026-03-28] Regra curta**
   Do instead: ação concreta e repetível

## Shell & Command Reliability
1. **[2026-03-28] Regra curta**
   Do instead: ação concreta e repetível

## User Directives
1. **[2026-03-28] Diretiva**
   Do instead: seguir exatamente
```

### O que entra no RUNBOOK

- Correções que o usuário fez durante a execução
- Comportamento inesperado de ferramentas ou comandos
- Desvios do REFINE que revelam um padrão
- Técnicas que funcionaram bem e devem ser repetidas

**O que não entra:** eventos únicos sem padrão, conclusões de tasks sem surpresas, logs de rotina.

Por volta da 3ª sessão, o agente começa a evitar erros antes que você precise corrigir.

---

## Test — *a verificação*

**Quando usar:** após o BUILD, antes do INTEGRATE.

```
/orbti:test                    # testa o refine mais recente
/orbti:test 4                  # testa a fase 4
/orbti:test 04-02              # testa refine específico
/orbti:test --manual           # pula detecção, vai direto para UAT manual
/orbti:test --e2e              # força Playwright (requer instalação)
```

### Como funciona

1. **Auto-detecta** o test runner do projeto (Jest, Vitest, pytest, go test...)
2. **Escreve testes** para ACs sem cobertura
3. **Executa** e mapeia resultados para cada AC
4. **Fallback** para UAT manual guiado se nenhum runner for encontrado

### E2E com Playwright

Playwright roda automaticamente se:
- `e2e.enabled: true` em `config.md`, **ou**
- `playwright-cli` declarado como required em `SPECIAL-FLOWS.md`

Com `--e2e` na flag, força independente do config.

**O que muda com Playwright habilitado:** todos os ACs são testados via browser, não apenas via unit/integration tests. O `/orbti:test` detecta a config e roteia automaticamente.

### Checklist de saída do TEST

```
════════════════════════════════════════
TEST RESULTS
════════════════════════════════════════

AC-1: Login funciona           ✓ PASS
AC-2: Token expira em 15min    ✓ PASS
AC-3: 401 em credenciais ruins ✗ FAIL — esperado 401, recebido 500

Issues: 1
Logged to: .orbti/projects/auth/01-UAT.md

════════════════════════════════════════
```

Problemas encontrados são logados no `UAT.md` para `/orbti:refine-fix`.

---

## Integrate — *fechar o loop*

**Nunca pule esta fase.** É obrigatória.

```
/orbti:integrate
/orbti:integrate .orbti/projects/auth/01-REFINE.md   # caminho específico
```

### O que o INTEGRATE registra

Cria `INTEGRATE.md` na mesma pasta do `REFINE.md` com:

| Seção | Conteúdo |
|---|---|
| **Performance** | Duração, tarefas completadas, arquivos modificados |
| **Acceptance Criteria Results** | Cada AC com PASS/FAIL |
| **Accomplishments** | O que foi entregue (substantivo, não genérico) |
| **Task Commits** | Hash de cada commit por tarefa |
| **Decisions Made** | Decisões tomadas durante execução com rationale |
| **Deviations** | Auto-fixados vs. deferidos, impacto |
| **Next Phase Readiness** | O que está pronto, preocupações, bloqueadores |

O frontmatter do INTEGRATE é machine-readable — usado para montar contexto automaticamente em futuras fases:

```yaml
---
phase: auth
refine: 01
subsystem: auth
tags: [jwt, bcrypt, express]
provides:
  - JWT login endpoint at POST /api/auth/login
  - User authentication with refresh tokens
affects: [dashboard, api-layer]
key-decisions:
  - "Chose jose library over jsonwebtoken for edge runtime compatibility"
---
```

Após o integrate, o STATE.md é atualizado:

```
REFINE ──▶ BUILD ──▶ INTEGRATE
  ✓        ✓        ✓
```

Se for o último refine de um projeto, o loop fecha automaticamente e atualiza o ROADMAP.

---

## ORBTI Skills — integrações externas

### O que são skills

Skills são workflows especializados que o ORBTI pode invocar durante o loop. Exemplo: ao construir UI, chamar `/figma:implement-design` antes do build. Ao criar uma API, chamar `/orbti:research` para documentação.

### Configurar skills do projeto

```
/orbti:skills            # configuração interativa completa
/orbti:skills add        # adicionar uma skill rapidamente
/orbti:skills list       # ver configuração atual
/orbti:skills audit      # checar se skills da fase atual foram invocadas
```

Isso cria `.orbti/SPECIAL-FLOWS.md`:

```markdown
| Work Type  | Skill/Command              | Priority | When Required         |
|------------|----------------------------|----------|-----------------------|
| UI work    | /figma:implement-design    | required | Before any UI build   |
| API design | /orbti:cocreate            | optional | For new endpoints     |
```

Skills marcadas como `required` **bloqueiam o BUILD** até serem carregadas. O REFINE.md inclui uma seção `<skills>` com checklist de confirmação.

### agentic-design — craft visual para projetos com UI

O ORBTI captura o Solution Intent (WHO/WHAT/FEEL) nativamente. Para projetos com interface visual, a skill **agentic-design** usa esse intent para criar um sistema de design consistente e evitar outputs genéricos.

```
https://github.com/Orbti-AI/orbti-design
```

**O que ela faz:**
- Cria `.orbti/design-system.md` com tokens (espaçamento, cores, tipografia, profundidade) derivados do Solution Intent
- Antes de cada componente: declara checkpoint de design (quem, o quê, como deve sentir)
- Após cada task de UI: revisa craft antes de mostrar o output (composição, espaçamento, hierarquia)
- Persiste decisões de design entre sessões via `design-system.md`

**Como configurar:**

1. Instale a skill:
```bash
npx github:Orbti-AI/agentic-design
```

2. Declare no projeto via SPECIAL-FLOWS.md:
```
/orbti:skills add
```

```markdown
| Work Type  | Skill/Command           | Priority | When Required              |
|------------|-------------------------|----------|----------------------------|
| UI screens | /agentic-design:init    | required | Antes do primeiro task de UI |
```

3. No BUILD, a skill é carregada automaticamente quando requerida. Ela lê o `CONTEXT.md` do projeto e usa o Solution Intent como base para as decisões visuais.

**Sem essa skill:** o ORBTI aplica o Solution Intent às decisões visuais via princípios básicos, mas sem sistema de tokens persistente nem revisão de craft automatizada.

---

### Instalar Playwright CLI como skill

O Playwright CLI é uma ferramenta externa — instale-a separadamente:

```bash
npm install -g @playwright/cli@latest
playwright-cli install --skills
playwright-cli install chromium
```

Então habilite no ORBTI:

```
/orbti:config → Playwright CLI E2E → Enable
```

O ORBTI verifica se o binário e a skill estão instalados antes de habilitar. Se algum faltar, mostra as instruções e não habilita.

Após habilitar, `/orbti:test` roda E2E automaticamente para todos os ACs — sem precisar de flag.

---

## Pausando e continuando o trabalho

O ORBTI persiste o estado entre sessões via `STATE.md`. Você pode parar a qualquer momento e retomar exatamente de onde parou.

```
/orbti:pause         # salva o estado atual e pausa o projeto ativo
/orbti:resume        # lista projetos em andamento e sugere próxima ação
/orbti:progress      # onde estou agora + próximo passo recomendado
```

**Fluxo típico de pausa e retomada:**

```bash
# ao terminar o dia (ou sessão)
/orbti:pause

# na próxima sessão
/orbti:resume        # lê STATE.md e handoffs automaticamente
                     # mostra: projeto ativo, fase atual, próximo comando
```

**O que o `resume` mostra:**
- Projeto e milestone ativos
- Último comando executado
- Próximo passo recomendado (ex: "pronto para /orbti:build")
- Desvios ou issues pendentes

O `STATE.md` é a fonte de verdade. Se o contexto parecer errado, cheque `.orbti/STATE.md` diretamente antes de continuar.

---

## Dicas do dia a dia

**Múltiplos projetos no mesmo milestone:**
```
/orbti:add-phase "nome"    # adicionar projeto ao milestone atual
/orbti:remove-phase        # remover projeto que ainda não começou
```

**Quando o build encontra issues para corrigir depois:**
```
/orbti:refine-fix   # planejar fixes de issues encontradas no /orbti:test
```

**Problemas comuns:**

| Problema | Solução |
|---|---|
| Comandos não encontrados após instalar | Reinicie o Claude Code. Verifique `~/.claude/commands/orbti/` |
| Loop position parece errado | Cheque `.orbti/STATE.md` diretamente. Rode `/orbti:progress` |
| Retomando após pausa | Rode `/orbti:resume` — lê STATE.md e handoffs automaticamente |
| Build falhou em uma tarefa | Use `[1] Retry` ou corrija o problema e rode `/orbti:build` novamente |

---

## O Fluxo Completo

```
                    ┌─────────────────────────────────────┐
                    │         OPCIONAL (pré-loop)         │
                    │                                     │
                    │  /orbti:observe-milestone           │
                    │    ↓                                │
                    │  /orbti:milestone "nome"            │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │         OPCIONAL (pré-fase)         │
                    │                                     │
                    │  /orbti:observe "projeto"       ← o quê     │
                    │    ↓                                         │
                    │  /orbti:cocreate "tema"         ← decisão   │
                    │    ↓                                         │
                    │  /orbti:research-phase "tema"   ← investigar│
                    │    ↓                                         │
                    │  /orbti:assumptions             ← validar   │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │            LOOP PRINCIPAL           │
                    │                                     │
                    │  /orbti:refine   ← plano + ACs      │
                    │    ↓                                │
                    │  /orbti:build    ← execução         │
                    │    ↓                                │
                    │  /orbti:test     ← verificação      │
                    │    ↓                                │
                    │  /orbti:integrate ← fechar loop     │
                    │    ↓                                │
                    │  (repetir para próximo refine)      │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │         FECHAR MILESTONE            │
                    │                                     │
                    │  /orbti:complete-milestone          │
                    └─────────────────────────────────────┘
```

**Versão compacta — um projeto típico:**

```bash
# setup único
/orbti:init
/orbti:config          # habilitar o que precisar

# opcional, quando há incógnitas
/orbti:observe "auth"
/orbti:cocreate "jwt vs session"
/orbti:research-phase "como auth está implementado hoje"

# loop — repita até o projeto completo
/orbti:refine
/orbti:build
/orbti:test
/orbti:integrate
```

---

## License

MIT. See [LICENSE](LICENSE).

This project is a fork of [CARL](https://github.com/ChristopherKahler/carl) by Chris Kahler.

---

<div align="center">

**Claude Code é poderoso. ORBTI torna previsível.**

</div>
