<purpose>
Sintetizar tudo que foi descoberto até agora (OBSERVE + RESEARCH + COCREATE) em um espelho compacto e verificável do contexto acumulado do projeto. O usuário lê em 30 segundos, confirma ou corrige — e o REFINE começa sem surpresas.

**Diferença do OBSERVE:** o OBSERVE captura o que o usuário quer. O assumptions mostra o que Claude entendeu e deduziu — e onde ainda há incerteza.
</purpose>

<process>

<step name="load_context" priority="first">
**Carregar todo o contexto acumulado do projeto.**

```bash
# Identificar projeto pelo argumento ou STATE.md
SLUG=$ARGUMENTS
cat .orbti/projects/${SLUG}/OBSERVE.md    2>/dev/null
cat .orbti/projects/${SLUG}/COCREATE.md   2>/dev/null
cat .orbti/projects/${SLUG}/RESEARCH.md   2>/dev/null
```

Se `$ARGUMENTS` vazio → usar projeto ativo do STATE.md:
```bash
cat .orbti/STATE.md 2>/dev/null | grep "Project:" | head -1
```

Se nenhum projeto identificado:
```
Error: projeto não encontrado.
Uso: /orbti:assumptions {slug}
```
→ PARAR.

**Coletar por fonte:**
- `OBSERVE.md` → goals, decisões estratégicas, riscos, solution intent
- `COCREATE.md` → plano por loop, decisões de cocriação, handoff
- `RESEARCH.md` → o que foi confirmado no codebase, perguntas respondidas, perguntas abertas
</step>

<step name="synthesize">
**Sintetizar em 4 blocos — do mais certo ao mais incerto.**

### Bloco 1 — O que sabemos (alta confiança)
Fatos confirmados: decisões fechadas no COCREATE, descobertas confirmadas no RESEARCH.
- Goals do projeto e loops planejados
- Decisões estratégicas já tomadas (ex: "reutilizar fila X", "nova coluna Y")
- O que o research confirmou no codebase (arquivos existentes, padrões, schemas)

### Bloco 2 — O que foi planejado (média confiança)
O plano existe mas ainda não foi validado tecnicamente pelo REFINE.
- Estrutura de loops e specialists
- Arquivos a criar/modificar (do handoff do COCREATE)
- Dependências entre loops

### Bloco 3 — O que ainda está aberto (baixa confiança / incerto)
Perguntas abertas do RESEARCH.md + HMW técnicos do handoff + o que não foi possível confirmar.
- HMW técnicos não respondidos
- Perguntas abertas do research
- Decisões que o REFINE precisa tomar

### Bloco 4 — O que Claude está assumindo (inferências)
Deduções razoáveis que NÃO estão explicitamente documentadas — onde o usuário deve corrigir se errado.
- Padrões inferidos (ex: "assumindo que o controller segue o padrão v2/carteira")
- Dependências implícitas
- Comportamentos esperados não documentados
</step>

<step name="present_and_save">
**Salvar ASSUMPTIONS.md na pasta do projeto e exibir ao usuário.**

Path: `.orbti/projects/{slug}/ASSUMPTIONS.md`

```markdown
---
project: {slug}
created: YYYY-MM-DD
sources: [OBSERVE.md, COCREATE.md, RESEARCH.md]
---

# ASSUMPTIONS — {slug}
{descrição do marco/contexto}

## O que sabemos
{bloco 1 — fatos confirmados, decisões fechadas}

## O que foi planejado
{bloco 2 — loops, specialists, handoff}

## O que ainda está aberto
{bloco 3 — HMW técnico, perguntas abertas}

## O que estou assumindo
{bloco 4 — inferências, marcadas como [inferência]}

---
*Gerado em {data} — corrija antes de /orbti:refine ou /orbti:build*
```

Exibir o conteúdo ao usuário e em seguida:

```
════════════════════════════════════════
ASSUMPTIONS salvo em .orbti/projects/{slug}/ASSUMPTIONS.md

Corrija o que estiver errado — as correções atualizam o arquivo.
▶ NEXT: /orbti:refine {slug} ou /orbti:build {refine}
════════════════════════════════════════
```

Aguardar resposta do usuário.
</step>

<step name="apply_corrections">
**Incorporar correções, atualizar ASSUMPTIONS.md e confirmar.**

Se usuário corrigiu algo:
1. Atualizar o arquivo `.orbti/projects/{slug}/ASSUMPTIONS.md` com as correções
2. Exibir:
```
Correções incorporadas em ASSUMPTIONS.md:
- [o que mudou]
```

Se usuário confirmou sem correções:
```
Contexto validado.
```

Oferecer próximo passo:
```
▶ NEXT: /orbti:refine {slug} ou /orbti:build {refine}
```
</step>

</process>

<success_criteria>
- [ ] Todo contexto acumulado lido (OBSERVE + COCREATE + RESEARCH)
- [ ] 4 blocos apresentados: sabemos / planejado / aberto / inferências
- [ ] Inferências claramente marcadas como [inferência]
- [ ] Usuário pode corrigir ou confirmar
- [ ] Próximo passo oferecido após validação
</success_criteria>
