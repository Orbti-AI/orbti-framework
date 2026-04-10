# Research — Ferramenta de Pesquisa Progressiva

O RESEARCH não é uma fase do loop — é uma **ferramenta transversal** que todas as fases usam. É o que permite que o entendimento do projeto se aprofunde progressivamente, da visão de negócio até o nível técnico de implementação.

---

## Como o Research Funciona

O research responde perguntas. A **profundidade da pesquisa é determinada pela profundidade da pergunta**.

Perguntas de negócio geram research de negócio.
Perguntas técnicas geram research técnico.

O ORBTI usa o mesmo mecanismo em todas as fases — o que muda é o que se está perguntando:

| Fase | Tipo de pergunta | Nível do research |
|------|-----------------|-------------------|
| OBSERVE | "Quais menus existem hoje? Que domínios são tocados?" | Negócio |
| COCREATE | "Como a tela X deve funcionar? O que precisa existir antes?" | Estratégico / UX |
| REFINE | "Em qual arquivo fica esse hook? Qual é o schema atual?" | Técnico |
| Manual | Qualquer pergunta a qualquer momento | Profundidade da pergunta |

---

## RESEARCH.md — O Arquivo que Acumula

Cada projeto tem um `RESEARCH.md` em `.orbti/projects/{slug}/`. Esse arquivo cresce ao longo do projeto, sendo enriquecido por cada fase:

```markdown
# RESEARCH — {Projeto}

## Nível de Negócio (Observe)
Preenchido pelo OBSERVE — domínios, menus, fluxos existentes

## Nível Estratégico (Cocreate)
Preenchido pelo COCREATE — UX, dependências, complexidade por loop

## Nível Técnico — Loop 1 (Refine)
Preenchido pelo REFINE — arquivos, hooks, schemas, endpoints

## Nível Técnico — Loop 2 (Refine)
Cada loop adiciona sua camada técnica
```

**Por que isso importa:** quando o REFINE do loop 3 rodar, ele já tem acesso ao research dos loops anteriores. Não repete descobertas — aprofunda nas questões novas daquele loop.

---

## Usando o Research Manualmente

Em qualquer momento, você pode pedir um research pontual:

```bash
/orbti:research {pergunta}
```

O nível de profundidade é detectado automaticamente pela natureza da pergunta:

```bash
# Pergunta de negócio → research de negócio
/orbti:research "Quais relatórios existem hoje no módulo de cessão?"

# Pergunta estratégica → research estratégico
/orbti:research "Como os usuários usam a tela de conciliação hoje?"

# Pergunta técnica → research técnico
/orbti:research "Como o hook useRetornoDavos está implementado?"
/orbti:research "Qual é o schema atual da tabela parcelas?"
```

O resultado é adicionado ao `RESEARCH.md` na seção correspondente ao nível da pergunta.

---

## Research Automático por Fase

### OBSERVE executa research de negócio automaticamente

Após a discussão de visão, o OBSERVE escaneia o codebase no nível de produto:
- Itens de menu/navegação que serão afetados
- Domínios de negócio envolvidos
- Fluxos existentes que mudarão

Nível: produto e negócio. Não desce para arquivos ou funções.

### COCREATE aprofunda para o nível estratégico

Após as perguntas HMW, o COCREATE aprofunda nas áreas reveladas:
- Como o usuário usa as telas hoje
- Dependências entre partes do sistema
- O que é simples vs complexo de construir (para ordenar os loops)

### REFINE aprofunda para o nível técnico

Antes de criar cada refine, o REFINE pesquisa no código:
- Arquivos específicos a modificar
- Componentes e hooks disponíveis
- Schemas, migrations e endpoints existentes
- Dependências entre refines do mesmo loop

---

## Qualidade do Research

O research tem três profundidades de execução:

| Profundidade | Quando | O que faz |
|-------------|--------|-----------|
| **Quick** | Verificar abordagem conhecida | Confirma que a abordagem ainda é válida |
| **Standard** | Escolha entre opções | Compara alternativas, produz recomendação |
| **Deep** | Problema novo, alta incerteza | Pesquisa exaustiva, verifica cada claim |

A profundidade é inferida pelo contexto ou pode ser especificada:

```bash
/orbti:research "deep: Como implementar retry com backoff no webhook processor?"
/orbti:research "quick: O hook useQuery do react-query ainda funciona assim?"
```

---

## Exemplo Prático — Projeto Eficiência de Débito

```
OBSERVE
  Pergunta: "Quais telas existem hoje no módulo de carteira?"
  Research: Escaneia navegação → encontra "Remessas", "Conciliação", "Fluxo"
  → RESEARCH.md: "## Nível de Negócio — navegação existente"

COCREATE
  Pergunta HMW: "Como poderíamos mostrar eficiência sem sobrecarregar a tela?"
  Research: Escaneia como usuário usa a tela hoje → identifica padrão de uso
  → RESEARCH.md: "## Nível Estratégico — padrão de uso das remessas"

REFINE (loop 1, Front)
  Pergunta: "Onde fica o componente de tabela que será estendido?"
  Research: Lê COMPONENTS.md → encontra <Table> em @/components/ui/table
  → RESEARCH.md: "## Nível Técnico — loop 1 — componentes disponíveis"

REFINE (loop 2, Front)
  Já tem o research anterior. Faz perguntas novas do loop 2.
  → RESEARCH.md: "## Nível Técnico — loop 2 — ..."
```

---

## Regras

| Regra | Detalhe |
|-------|---------|
| Não repete research | Cada fase lê o RESEARCH.md antes de pesquisar — não refaz o que já foi feito |
| Acumula, não substitui | Cada camada é adicionada ao RESEARCH.md, não sobrescreve |
| Silencioso por padrão | Research automático (OBSERVE, COCREATE, REFINE) não anuncia cada busca — só o resultado |
| Profundidade pela pergunta | Não há configuração de nível — a pergunta determina o que é necessário pesquisar |
