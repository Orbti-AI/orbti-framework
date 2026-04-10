# RESEARCH — Documento de Enquadramento Progressivo

O RESEARCH não é uma fase do loop — é um **documento vivo** que o OBSERVE e o COCREATE
preenchem progressivamente, e que o REFINE consome como base de trabalho.

**Propósito central:** dar enquadramento e limites ao REFINE para que ele não extrapole o
escopo, não reabra decisões já tomadas, e não desperdice tokens re-pesquisando o que já foi descoberto.

---

## O Objetivo do RESEARCH.md

O REFINE lê o RESEARCH.md **antes de qualquer pesquisa técnica**. Ao terminar a leitura, ele deve saber:

1. **O que é o projeto** — contexto de negócio suficiente para não precisar abrir o OBSERVE.md
2. **O que já foi decidido** — decisões do COCREATE que não devem ser reabertas
3. **Os limites do loop** — o que entra e o que **não** entra no escopo
4. **O que aprofundar** — perguntas específicas que o REFINE ainda deve investigar

> **Meta:** OBSERVE.md e COCREATE.md devem se tornar dispensáveis para o REFINE.
> O RESEARCH é denso o suficiente para substituí-los no contexto de planejamento.

---

## Quem Preenche e Quando

```
OBSERVE  → preenche "Enquadramento de Negócio"
             regras, fluxos, o que existe, o que está fora do escopo

COCREATE → preenche "Decisões Estratégicas"
             o que foi decidido e não deve ser reaberto

REFINE   → preenche "Contexto Técnico Acumulado" (por loop)
             o que foi confirmado tecnicamente — o próximo REFINE parte daqui
```

O BUILD não lê o RESEARCH — lê o REFINE, que já tem tudo que precisa para execução.

---

## Estrutura do RESEARCH.md

```markdown
# RESEARCH — {Projeto}

> Para o REFINE: leia antes de qualquer pesquisa. OBSERVE.md e COCREATE.md
> são dispensáveis depois daqui.

## Enquadramento de Negócio `(OBSERVE)`
Regras de negócio, fluxos, o que já existe, o que está fora do escopo.
Denso o suficiente para substituir o OBSERVE.md no contexto do REFINE.

## Decisões Estratégicas `(COCREATE)` — Não Reabrir
Tabela: decisão | o que não propor como alternativa.

## Limites por Loop — Escopo e Fronteiras
Para cada loop:
- O que entrega
- O que NÃO extrapolar
- Perguntas para o REFINE aprofundar (algumas viram HMW, outras são pesquisa direta)

## Contexto Técnico Acumulado
Por loop — preenchido pelo REFINE após pesquisa técnica.
O próximo REFINE parte daqui sem re-pesquisar.
```

---

## As Perguntas para Aprofundar

A seção "Perguntas para o REFINE aprofundar" é onde o RESEARCH indica o que ainda está
em aberto — sem tentar responder, porque responder é papel do REFINE após pesquisa técnica.

Cada pergunta é classificada implicitamente em dois tipos:

**Pesquisa direta no código** — há um arquivo ou query que responde:
> "Migration `enviado_automaticamente` foi aplicada?" → verificar `supabase/migrations/`

**HMW técnico** — há uma decisão de design que o REFINE deve tomar após pesquisar:
> "Como agrupar por banco sem criar acoplamento com a estrutura V1?" → REFINE pesquisa e propõe

O REFINE usa essas perguntas como ponto de entrada do research técnico — não começa do zero,
começa de onde o RESEARCH parou.

---

## O que Não Vai no RESEARCH.md

| O que evitar | Por quê |
|-------------|---------|
| Comandos grep ou bash | São detalhe de execução — pertencem ao REFINE |
| Pesquisa técnica de arquivos específicos | É papel do REFINE, não do RESEARCH |
| Especulação sobre o que o BUILD vai encontrar | O BUILD lê o REFINE, não o RESEARCH |
| Repetição do OBSERVE.md ou COCREATE.md na íntegra | Deve resumir e sintetizar, não copiar |

---

## Regras

| Regra | Detalhe |
|-------|---------|
| OBSERVE e COCREATE preenchem | O REFINE consome — nunca o inverso |
| Acumula por loop | Cada REFINE adiciona sua camada técnica — não sobrescreve |
| Denso, não extenso | Substitui os documentos de origem — não os duplica |
| Perguntas abertas, não respostas | O que aprofundar é indicado, não respondido antecipadamente |
| Silencioso durante as fases | OBSERVE e COCREATE preenchem sem anunciar cada busca |
