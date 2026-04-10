# Project Config Template

Template for `.orbti/config.md` - project-specific configuration and integrations.

**Purpose:** Store project settings and optional integration flags. Allows ORBTI to adapt behavior based on available tools.

---

## File Template

```markdown
# Project Config

**Project:** [project-name]
**Created:** [YYYY-MM-DD]

## Project Settings

```yaml
project:
  name: [project-name]
  version: [current-version or "0.0.0"]
```

## Integrations

Optional integrations that enhance ORBTI functionality.

### E2E Testing (Playwright)

```yaml
e2e:
  enabled: false
  base_url: http://localhost:3000
```

### SonarQube

Code quality scanning and static analysis.

```yaml
sonarqube:
  enabled: false
  project_key: [project-key]  # Must match sonar-project.properties
  server_url: http://localhost:9000  # Optional, for reference
```

**When enabled:**
- `/orbti:quality-gate` runs scans and updates CONCERNS.md
- Quality metrics inform planning decisions
- Issues feed into tech debt tracking

**Requirements:**
- SonarQube server running (local or cloud)
- `sonar-project.properties` in project root
- SonarQube MCP server configured

### E2E Testing (Playwright)

Browser-based end-to-end testing against acceptance criteria.

```yaml
e2e:
  enabled: false
  base_url: http://localhost:3000   # App URL for Playwright to navigate
```

**When enabled:**
- BUILD finalize auto-runs Playwright for ALL ACs (no `--e2e` flag needed)
- `checkpoint:human-verify` executes automatically — Playwright navigates, interacts, screenshots
- Human verification still runs after E2E, focused on UX/visual judgment only
- E2E failures are warnings (not blockers) — flakiness is real

**Requirements:**
- Playwright installed: `npx playwright --version`
- App must be running at `base_url` before BUILD starts
- If app is not running, BUILD presents `checkpoint:human-action` to start it

### Future Integrations

Reserved for future use:

```yaml
# linting:
#   enabled: false
#   config_file: .eslintrc

# testing:
#   enabled: false
#   coverage_threshold: 80
```

### Design (orbti:design)

Habilita o special flow de design: após `/orbti:observe`, se há interfaces a construir, sugere rodar `/orbti:design` para criar protótipos antes do planejamento.

```yaml
design:
  enabled: false
  prototype_tool: paper   # paper | pencil | figma | stitch | html
  auto_suggest: true      # true = sugere automaticamente no handoff do observe
```

**Quando enabled:**
- `/orbti:observe` detecta interfaces no CONTEXT.md e oferece rodar `/orbti:cocreate` em design mode
- Protótipos são gerados com a ferramenta definida em `prototype_tool`
- DESIGN.md + system.md são produzidos e servem de referência para `/orbti:refine`
- `/orbti:refine` lê os protótipos e gera ACs visuais específicas

**Ferramentas disponíveis:**
- `paper` — Paper.design (MCP: `mcp__paper__*`) — nativo HTML/CSS, ideal para iteração rápida
- `pencil` — Pencil (MCP: `mcp__pencil__*`) — arquivos .pen, componentes visuais
- `figma` — Figma (MCP: `mcp__plugin_figma_figma__*`) — implementar designs existentes
- `stitch` — Google Stitch — geração visual standalone sem MCP
- `html` — HTML puro — zero setup, sem MCP necessário

**Requisitos:**
- MCP da ferramenta conectado (exceto `html` e `stitch`)
- `.orbti-design/` diretório criado (gerado automaticamente pelo orbti:design)

## Models

Model routing for ORBTI phases. See `./.claude/orbti-framework/references/model-routing.md` for defaults.

```yaml
models:
  default: sonnet             # Fallback for any phase not listed
  overrides: {}               # Override per-phase: refine: opus, build: haiku, etc.
```

## Parallel Build (Agent Teams)

Executa múltiplos REFINEs simultaneamente quando estão no mesmo loop sem dependências entre si.
Requer `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` no ambiente.

```yaml
parallel_build:
  enabled: false   # true = Agent Teams paralelo | false = sequencial (padrão)
```

**Quando enabled:**
- BUILD detecta refines na mesma loop sem depends_on pendentes
- Spawna um teammate por refine independente
- Refines com depends_on aguardam a task bloqueante completar antes de iniciar
- Checkpoints (human-verify, decision) pausam o teammate e sobem ao lead
- Fallback automático para sequencial se env var não estiver setada

**Requisito:** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` em `./.claude/settings.json`:
```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

## Preferences

Optional user preferences for ORBTI behavior.

```yaml
preferences:
  auto_commit: false          # Auto-commit after successful tasks
  verbose_output: false       # Show detailed step output
```

---

*Config created: [timestamp]*
*Edit anytime - changes take effect on next command*
```

---

## Good Example

```markdown
# Project Config

**Project:** expense-tracker
**Created:** 2026-01-28

## Project Settings

```yaml
project:
  name: expense-tracker
  version: 0.2.0
```

## Integrations

### SonarQube

```yaml
sonarqube:
  enabled: true
  project_key: expense-tracker
  server_url: http://localhost:9000
```

### Future Integrations

```yaml
# linting:
#   enabled: false
```

## Preferences

```yaml
design:
  enabled: true
  prototype_tool: paper
  auto_suggest: true

parallel_build:
  enabled: false

preferences:
  auto_commit: false
  verbose_output: false
```

---

*Config created: 2026-01-28*
```

---

## Guidelines

**What belongs in config.md:**
- Project identification (name, version)
- Integration toggles (enabled/disabled flags)
- Integration-specific settings (project keys, URLs)
- User preferences for ORBTI behavior

**What does NOT belong here:**
- Sensitive credentials (use environment variables)
- Build configuration (use native config files)
- Project requirements (that's PROJECT.md)
- Roadmap information (that's ROADMAP.md)

**When to create config.md:**
- During `/orbti:init` if user enables integrations
- Manually when adding integrations later
- Not required for basic ORBTI usage

**Git behavior:**
- Can be committed (no secrets)
- Or gitignored if preferences are personal
- Recommend: commit integration flags, gitignore preferences
