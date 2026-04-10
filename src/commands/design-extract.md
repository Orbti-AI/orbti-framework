---
name: orbti:design-extract
description: Extrai padrões de design do código existente do CRM e salva em .orbti-design/system.md. Use quando quiser sincronizar o design system real com o orbti-design.
argument-hint: "[caminho opcional de componentes]"
allowed-tools: [Read, Glob, Grep, Write, Edit]
---

<objective>
Extrair padrões de design do código existente do CRM — tokens, componentes, espaçamentos, tipografia — e salvar em `.orbti-design/system.md` para que futuras sessões de design partam do design system real do projeto.
</objective>

<model>sonnet</model>

<execution_context>
@.claude/skills/orbti-design/SKILL.md
</execution_context>

<process>

## Passo 1 — Ler tokens oficiais

Ler o arquivo de tokens configurado em `.orbti-design/config.json`:
```bash
cat .orbti-design/config.json
```

Ler o arquivo de tokens encontrado (ex: `apps/crm-menos-juros/client/src/styles/design-tokens.css`).

Extrair:
- Paleta de cores (background, foreground, border, brand, semantic)
- Tipografia (font-family, sizes, weights)
- Espaçamento (escala base)
- Border radius
- Sombras

## Passo 2 — Mapear componentes existentes

Ler `components_path` do `.orbti-design/config.json`. Buscar componentes UI reutilizáveis nessa pasta:

```bash
find {components_path} -name "*.tsx" | sort
```

Listar TODOS os componentes encontrados. Para cada componente relevante (Button, Card, Badge, Table, Input, Modal, Alert, Sheet, Dialog, Sidebar, DataTable, Select, Tabs, Progress, Skeleton, Spinner, Tooltip, Popover):
- Ler o arquivo
- Extrair: dimensões, padding, radius, variantes, estados
- Registrar no system.md com localização: `{components_path}/ui/{nome}.tsx`

Incluir no system.md uma seção **Inventário de Componentes** listando todos os `.tsx` encontrados em `{components_path}/ui/` — facilita saber o que já existe sem ter que buscar a cada sessão.

## Passo 3 — Identificar padrões de uso

Buscar padrões recorrentes no código:
- Espaçamentos mais usados
- Variantes de componentes
- Hierarquia tipográfica real

## Passo 4 — Gerar system.md

Escrever `.orbti-design/system.md` com o que foi encontrado:

```markdown
# Design System — CRM Menos Juros

*Extraído em: {data}*
*Fonte: {caminho dos tokens}*

## Tokens de Cor

### Background
{tokens extraídos}

### Foreground / Texto
{tokens extraídos}

### Border
{tokens extraídos}

### Semântico
{tokens extraídos (destructive, warning, success, muted)}

## Tipografia

{font-family, tamanhos, pesos}

## Espaçamento

{escala base e valores}

## Border Radius

{escala de radius}

## Sombras / Depth

{estratégia adotada: borders-only | subtle-shadows | surface-shifts}

## Componentes

### {Nome do Componente}
{dimensões, padding, variantes, estados}

## Padrões de Layout

{sidebar, cards, tabelas — o que existe hoje}

## Notas

{observações sobre o design system — inconsistências, padrões a evitar}
```

## Passo 5 — Confirmar

Exibir resumo do que foi extraído e perguntar se quer salvar.

```
════════════════════════════════════════
DESIGN SYSTEM EXTRAÍDO
════════════════════════════════════════

Tokens encontrados: {n cores, tipografia, espaçamento}
Componentes mapeados: {lista}
Salvo em: .orbti-design/system.md

A partir de agora, /orbti:design-prototype e
/orbti:cocreate usarão esse design system.
════════════════════════════════════════
```

</process>
