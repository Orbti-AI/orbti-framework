# COCREATE.md Template

Template for `.orbti/projects/{slug}/COCREATE.md` — strategic co-creation document.

**Purpose:** Materializar a visão do observe em um plano estratégico por specialist e por loop. Nível: estratégico (não técnico).

**Pré-requisito:** OBSERVE.md deve existir na pasta do projeto.

---

## File Template

```markdown
---
project: {slug}
created: YYYY-MM-DD
observe: OBSERVE.md
loops: N
---

# COCREATE — {Tópico}

## Visão Materializada
[Síntese da visão desejada — mais concreta que o observe, menos técnica que o refine.
O que será verdade quando o projeto estiver pronto, do ponto de vista do usuário.]

## HMW — Perguntas de Cocriação

1. Como poderíamos {X} para que {usuário/role} consiga {resultado esperado}?
2. Como poderíamos {X} sem {trade-off indesejado identificado no observe}?
3. Como poderíamos {X} mesmo que {restrição ou risco do OBSERVE.md}?
4. Como poderíamos garantir {Y} considerando {contexto específico do projeto}?
5. Como poderíamos {X} de forma que {feel do Solution Intent do OBSERVE.md}?

## Research Aprofundado
[Descobertas além do OBSERVE.md — nível de experiência do usuário e planejamento estratégico.
Não inclui arquivos específicos ou funções — isso é papel do refine.]

- [Área de UX]: [descoberta sobre como o usuário usa / deve usar]
- [Dependência estratégica]: [o que precisa existir antes de quê]
- [Complexidade]: [o que é simples vs complexo de construir]

## Plano por Loop e Specialist

### Loop 1 — [Descrição do objetivo do loop]

#### Front (F)
O que será construído ou alterado no frontend:
- [Tela/componente]: [como funciona — o que mostra, interações, estado vazio, ação primária]
- [Navegação]: [alterações no menu, rotas ou breadcrumbs]
- [Reutilizar]: [o que já existe e será mantido]

#### Back (B)
O que será alterado ou construído no backend:
- [Endpoint/regra]: [descrição — dados, comportamento]
- [O que muda vs o que é novo]

#### Agents (A) [omitir se não aplicável]
- [Automação] → trigger: [X], ação: [Y], resultado: [Z]

#### Tests (T)
Cenários de validação deste loop:
- [Cenário]: [o que valida, resultado esperado]

---

### Loop 2 — [Descrição]

#### Front (F)
...

#### Back (B)
...

---

## Protótipos [adicionar após /orbti:design, se executado]
- [Tela X]: [tool usado — arquivo/artboard/URL]
- [Tela Y]: [tool usado — arquivo/artboard/URL]

## Decisões de Cocriação
Escolhas estratégicas feitas — com rationale:
- [Decisão]: [por que essa escolha e não outra]

## Handoff para Refine

> Leitura obrigatória do /orbti:refine antes de criar qualquer task.
> Objetivo: reaproveitamento máximo de contexto — o refine não re-descobre o que o cocreate já resolveu.

### Loop 1 — [Descrição]

**Leia antes de criar tasks:**
- `[arquivo/path]` — [o que verificar / por que importa]
- `[outro arquivo]` — [o que confirmar]

**HMW técnico (abrir no início do refine):**
- [Decisão a tomar]: [contexto — o que já se sabe, opções conhecidas]
- [Outra decisão]: [contexto]

**Riscos a endereçar nas tasks:**
- [Risco]: [como mitigar / o que verificar]

**Protótipos para este loop** (se design executado):
- [Tela X]: [referência — arquivo .pen / artboard Paper / URL Figma]

---

### Loop 2 — [Descrição]

**Leia antes de criar tasks:**
- `[arquivo]` — [por que importa]

**HMW técnico:**
- [Decisão]: [contexto]

**Riscos:**
- [Risco]: [mitigação]

**Protótipos:**
- [Tela Y]: [referência]
```

---

## Regras de Preenchimento

### Seção Front (F)

Especificar como as telas funcionam — não só o que existe:

| Elemento | O que especificar |
|----------|------------------|
| Tela | O que mostra, como organiza a informação |
| Filtros | Quais filtros, comportamento padrão |
| Ação primária | O que acontece ao executar |
| Estado vazio | O que o usuário vê quando não há dados |
| Estado de erro | Como erros são apresentados |
| Navegação | Como o usuário chega e sai desta tela |

### Seção Back (B)

Especificar em nível de domínio — não de implementação:

- O que a operação faz (verbo + objeto)
- Dados que entram e saem
- Regras de negócio envolvidas
- O que muda vs o que é construído do zero

### HMW (How Might We)

Cada pergunta deve ser:
- Específica ao projeto (não genérica)
- Orientada a um ângulo diferente (usuário, técnica, restrição, risco, feel)
- Aberta o suficiente para ter mais de uma resposta possível

---

## Campos do Frontmatter

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| `project` | Sim | Slug do projeto |
| `created` | Sim | Data de criação |
| `observe` | Sim | Arquivo de observe usado como input |
| `loops` | Sim | Número de loops planejados |
