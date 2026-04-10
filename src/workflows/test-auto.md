<purpose>
Criar o TEST-PLAN.md do refine, executar testes automatizados disponíveis e mapear resultados por AC.

`/orbti:test` é o único responsável por escrever e executar testes — BUILD não escreve testes.

**Responsabilidades:**
- Criar `{refine}-TEST-PLAN.md` documentando cenários, tipos e checkpoints
- Detectar e executar runner do projeto (unit/integration)
- Mapear resultados por AC: PASS / FAIL / MANUAL
- Atualizar TEST-PLAN.md com status final
- Rotear para INTEGRATE (pass) ou refine-fix (fail)

**Fora do escopo deste workflow:**
- Testes E2E/browser — se o projeto declarar uma skill de E2E em SPECIAL-FLOWS.md, invocar essa skill passando o TEST-PLAN.md como entrada
- Playwright, Cypress ou qualquer runner de browser — não invocar diretamente
</purpose>

<test_runner_detection>
Detect test runner by inspecting project files — use what's already there, install nothing new.

```bash
# Node / JavaScript
cat package.json 2>/dev/null | grep -E '"test"|"jest"|"vitest"|"mocha"'

# Python
ls pytest.ini pyproject.toml setup.cfg conftest.py 2>/dev/null

# Go
ls go.mod 2>/dev/null && find . -name "*_test.go" -not -path "*/vendor/*" | head -3

# Rust
ls Cargo.toml 2>/dev/null && grep -r "#\[test\]" src/ --include="*.rs" | head -3

# Other
ls Makefile 2>/dev/null && grep -E "^test:" Makefile
```

**Decision matrix:**

| Found | Runner | Run command |
|-------|--------|-------------|
| `jest` in package.json | Jest | `npx jest --testPathPattern=<pattern>` |
| `vitest` in package.json | Vitest | `npx vitest run` |
| `pytest.ini` or `conftest.py` | Pytest | `python -m pytest` |
| `*_test.go` files | Go test | `go test ./...` |
| `#[test]` in Rust | Cargo | `cargo test` |
| `test:` in Makefile | Make | `make test` |
| None found | — | Fall back to manual UAT |

</test_runner_detection>

<process>

<step name="create_test_plan" priority="first">
**Criar TEST-PLAN.md antes de qualquer execução de testes.**

Este arquivo é o contrato de testes do refine. Deve ser criado SEMPRE — mesmo antes de rodar um único teste.

**Localização:** `.orbti/projects/{projeto}/{refine}-TEST-PLAN.md`

**Como construir:**

1. Ler o REFINE.md do escopo identificado
2. Para cada AC, mapear:
   - Cenário principal (happy path)
   - Cenários de edge case relevantes
   - Tipo de teste: `unit` | `integration` | `e2e` | `manual`
   - Checkpoint necessário: `auto` (roda sem humano) | `human` (precisa de aprovação)
   - Dados necessários (seletores, fixtures, usuário de teste, etc.)

**Template do arquivo:**

```markdown
---
project: {projeto}
refine: {N}
type: test-plan
status: pending   ← muda para: running | passed | failed
date: {data}
---

# TEST-PLAN: {nome do refine}

## Escopo
Testando: {descrição de 1 linha do que foi construído}
Arquivo(s) modificado(s): {lista}

## Cenários por AC

### AC-1: {título}
| # | Cenário | Tipo | Checkpoint | Dados necessários |
|---|---------|------|------------|-------------------|
| 1 | Happy path: {descrição} | e2e | auto | {seletor, url, usuário} |
| 2 | Edge case: {descrição} | unit | auto | {mock, fixture} |
| 3 | Erro esperado: {descrição} | manual | human | {passos manuais} |

### AC-2: {título}
...

## Checkpoints Humanos
Lista de checks que requerem validação humana (tipo: `human`):
- [ ] AC-1 cenário 3: {o que verificar}
- [ ] AC-2 cenário 2: {o que verificar}

## Infraestrutura de Testes
- Runner detectado: {vitest | jest | playwright | manual}
- Base URL: {se E2E}
- Usuário de teste: {se necessário}
- Fixtures/mocks: {se necessário}

## Resultado (preenchido após execução)
| AC | Cenário | Status | Observação |
|----|---------|--------|------------|
| AC-1 | Happy path | ⏳ | — |

## Veredicto final
Status: PENDING → PASS | FAIL | PARTIAL
```

Salvar o arquivo com `status: pending` antes de começar os testes.
Atualizar `status` e a tabela de Resultado durante e após execução.
</step>

<step name="detect_autonomous_mode">
**Verificar se está rodando dentro de um loop worktree (autonomous).**

```bash
grep "worktree_branch:" .orbti/projects/*/*.md 2>/dev/null | grep -v "^Binary" | head -1
```
**Verificar se está rodando dentro de um loop worktree (autonomous).**

```bash
grep "worktree_branch:" .orbti/projects/*/*.md 2>/dev/null | grep -v "^Binary" | head -1
```

Ou verificar no REFINE.md identificado:
```bash
grep "worktree_branch:" [refine-path]
```

**Se `worktree_branch` presente no REFINE.md → `autonomous_mode: true`**

Impactos do `autonomous_mode: true`:
- Checkpoints manuais são **pulados** — não há humano disponível
- Cenários `manual` no TEST-PLAN ficam com status `skipped` (não bloqueiam)
- Skill de E2E declarada em SPECIAL-FLOWS → invocar automaticamente se disponível
- Todos os resultados são logados no TEST-PLAN e o fluxo reporta ao lead via SendMessage

**Se `worktree_branch` ausente → `autonomous_mode: false` (fluxo interativo com checkpoints)**
</step>

<step name="identify_scope">
**Determine what to test:**

If $ARGUMENTS provided: find corresponding REFINE.md and INTEGRATE.md
If not: use most recently modified REFINE.md

Read REFINE.md to extract:
- All acceptance criteria (AC-1, AC-2, ...)
- Files created/modified (from INTEGRATE.md if exists)
- Boundaries (what was NOT changed)
</step>

<step name="check_custom_skill">
**Verificar se projeto tem skill de teste declarada em SPECIAL-FLOWS.md:**

```bash
grep -i "test\|spec\|testes" .orbti/SPECIAL-FLOWS.md 2>/dev/null | grep -o '`[^`]*`' | tr -d '`'
```

Procurar nas linhas da tabela Project-Level Skills que mencionem "test", "spec" ou "testes".
Extrair o nome da skill da coluna Skill (formato: `nome-da-skill`).

**Se encontrar:** invocar a skill → ela substitui este workflow inteiro.
A skill pode declarar `tdd: true` no seu output — nesse caso:
- Os testes são escritos para FALHAR (red) antes do BUILD
- O BUILD lê SPECIAL-FLOWS, detecta TDD, e executa para fazê-los passar (green)
- O `/orbti:test` pós-BUILD confirma que estão passando

**Se não encontrar:** continuar com o agente padrão abaixo (cobre casos básicos).
</step>

<step name="detect_runner">
**Auto-detect test runner (unit/integration only):**

Run the detection commands above.

**Decision:**
- Runner encontrado → escrever e executar testes automatizados
- Runner não encontrado → fallback para manual UAT

**E2E / browser tests:**
Verificar se projeto declara skill de E2E em SPECIAL-FLOWS.md:
```bash
grep -i "e2e\|playwright\|cypress\|browser" .orbti/SPECIAL-FLOWS.md 2>/dev/null | head -3
```
Se declarar uma skill de E2E:
- Criar o TEST-PLAN.md normalmente (com cenários E2E documentados como tipo `e2e`)
- Após criar TEST-PLAN.md, invocar a skill de E2E declarada passando o caminho do TEST-PLAN como argumento
- A skill de E2E lê o TEST-PLAN e executa os cenários `e2e`
- NÃO invocar Playwright, Cypress ou qualquer runner de browser diretamente neste workflow

Se não declarar skill de E2E:
- Cenários de tipo `e2e` ficam como `manual` no TEST-PLAN
- Usuário executa manualmente e confirma resultado

Se nenhum runner encontrado:
```
Nenhum runner detectado. Usando UAT manual.
```
→ Switch to: @./.claude/orbti-framework/workflows/verify-work.md
</step>

<step name="write_tests">
**Escrever testes para cada AC do REFINE.md:**

## Descobrir convenção de pasta

**1. Verificar se projeto tem `.orbti/TESTS.md`:**
```bash
cat .orbti/TESTS.md 2>/dev/null
```

Se existir → usar as configurações definidas lá (módulos, runners, paths, aliases).

Se não existir → usar convenção padrão:
```
tests/
  {feature}/    ← mesma pasta do runner detectado na raiz
```
E imports relativos ao runner detectado.

**2. Para cada AC extraído:**
1. Identificar qual módulo é afetado (ler `TESTS.md` — coluna `module` ou `affects`)
2. Escrever arquivo no path configurado para aquele módulo
3. Nome de arquivo descritivo: `buscar-pendentes.use-case.spec.ts`, `dashboard-sort.test.ts`
4. Usar o import alias definido em `TESTS.md` (ex: `src/`, `@/`, relativo)
5. Testar comportamento (não implementação interna)
6. Sem novas dependências além das já instaladas

Verificar se algum teste já existe antes de escrever (evitar duplicata):
```bash
ls .orbti/TESTS.md 2>/dev/null && \
  grep -r "AC-[0-9]" $(grep "path:" .orbti/TESTS.md | awk '{print $2}' | head -1) 2>/dev/null || \
  grep -r "AC-[0-9]" tests/ 2>/dev/null
```
</step>

<step name="run_tests">
**Run the test suite:**

Run only tests related to the current refine's scope when possible.
If the runner supports file filtering, prefer that over running the full suite.

Capture full output. Look for:
- Pass / fail counts
- Which specific tests failed
- Error messages for failures
</step>

<step name="map_results_to_acs">
**Map test results back to ACs:**

Build a results table:

```
AC-1: [description] ........... PASS
AC-2: [description] ........... FAIL — [error summary]
AC-3: [description] ........... PASS
```

For each FAIL:
- Extract the error message
- Note which test file and line
- Categorize: assertion error / exception / timeout / setup failure
</step>

<step name="report_and_route">
**Apresentar resultados, atualizar TEST-PLAN.md e rotear.**

**Gate obrigatório: evidência antes de qualquer veredicto.**
Mostrar o output real do runner (comando executado + saída completa com pass/fail count) antes de declarar qualquer AC como PASS.
Não afirmar "todos passaram" sem mostrar o output. "O runner não deu erro" não é evidência — é ausência de evidência.

```
════════════════════════════════════════
TEST RESULTS: [Nome do Refine]
════════════════════════════════════════

Runner: [detectado]
TEST-PLAN: .orbti/projects/{projeto}/{refine}-TEST-PLAN.md

AC-1: ✓ PASS — [cenário]
AC-2: ✗ FAIL — [erro resumido]
AC-3: ○ MANUAL — aguarda validação humana

Automatizados: [total] | Passou: [N] | Falhou: [N]
Manuais pendentes: [N]

════════════════════════════════════════
Veredicto: [ALL PASS | FAILURES FOUND | MANUAL PENDING]
════════════════════════════════════════
```

**Atualizar TEST-PLAN.md:**
- Preencher tabela de Resultado com status por cenário
- Atualizar `status:` no frontmatter:
  - Todos passando → `passed`
  - Algum falhou → `failed`
  - Testes automatizados passaram mas há manuais pendentes → `partial`

**Cenários `manual` no TEST-PLAN:**
Apresentar cada um como checkpoint ao usuário:
```
════════════════════════════════════════
CHECKPOINT MANUAL: AC-{N} — {título}
════════════════════════════════════════
{passos descritos no TEST-PLAN}

Resultado: [pass / fail / skip]
════════════════════════════════════════
```
Registrar resultado no TEST-PLAN.

**Se skill de E2E declarada em SPECIAL-FLOWS e cenários `e2e` existem no TEST-PLAN:**
```
Cenários E2E identificados no TEST-PLAN.
Invocar skill de E2E com: {caminho do TEST-PLAN}
Aguardar resultado da skill → registrar no TEST-PLAN.
```

**Roteamento final:**

| Veredicto | Ação |
|-----------|------|
| `passed` | Atualizar STATE.md → TEST ✓ → informar que INTEGRATE está liberado |
| `partial` | Perguntar: continuar com manuais pendentes ou resolver primeiro |
| `failed` | Oferecer `/orbti:refine-fix` — INTEGRATE bloqueado até resolver |

**Se FAILURES FOUND:**
- Logar falhas em `.orbti/projects/{projeto}/{refine}-UAT.md`
- Oferecer: "Rodar /orbti:refine-fix para corrigir antes do INTEGRATE?"
- Usuário pode aprovar INTEGRATE mesmo assim (falhas ficam documentadas no INTEGRATE.md)
</step>

</process>

<success_criteria>
- [ ] TEST-PLAN.md criado em `.orbti/projects/{projeto}/{refine}-TEST-PLAN.md`
- [ ] Cenários mapeados por AC com tipo (unit/integration/e2e/manual) e checkpoint
- [ ] Custom skill de E2E verificada em SPECIAL-FLOWS.md (invocar se existir)
- [ ] Test runner detectado (ou fallback manual ativado)
- [ ] Testes automatizados escritos e executados
- [ ] Resultados mapeados por AC: AC-1 PASS/FAIL, AC-2 PASS/FAIL...
- [ ] TEST-PLAN.md atualizado com status final (passed/failed/partial)
- [ ] Issues logadas em `{refine}-UAT.md` se falhas
- [ ] STATE.md atualizado: TEST ✓ ou TEST ✗
- [ ] Usuário roteado para INTEGRATE (passed) ou refine-fix (failed)
</success_criteria>
