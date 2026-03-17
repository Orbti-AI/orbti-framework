<objective>
Manage ORBIT project configuration and integrations. Create or update .orbit/config.md at any point in the project lifecycle.
</objective>

<when_to_use>
- Enable SonarQube after project init
- Disable an integration
- View current configuration
- Change project settings
</when_to_use>

<process>

**Step 1: Check for existing config**

```bash
ls .orbit/config.md 2>/dev/null
```

**If config exists:**
```
Current configuration:

[Display config.md contents]

What would you like to do?
[1] Enable/disable integration
[2] View full config
[3] Reset to defaults
```

**If config doesn't exist:**
```
No configuration found.

Would you like to set up project configuration?
[1] Yes, create config
[2] Cancel
```

**Step 2: Handle user choice**

**For new config or "Enable/disable integration":**

```
Available integrations:

[1] SonarQube - Code quality scanning
    Status: [enabled/disabled/not configured]

[2] Playwright CLI E2E - Browser-based acceptance testing (default: disabled)
    Status: [enabled/disabled/not configured]

[3] Parallel Tests - Write tests during BUILD using agent teams (default: disabled)
    Status: [enabled/disabled] (requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)

[4] Agent Teams - Parallel research and review
    Status: [enabled/disabled] (requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)

[5] Done - save and exit
```

**If user selects SonarQube:**

Prompt for project_key (default: directory name).
Set sonarqube.enabled = true/false in config.md.

---

**If user selects Playwright CLI E2E:**

```
Playwright CLI E2E — run browser tests on /orbit:test
Current: [enabled/disabled]

[1] Enable  [2] Disable  [3] Back
```

If enabling: set e2e.enabled = true in config.md.
If disabling: set e2e.enabled = false.

Note: requires Playwright CLI installed. Run /orbit:enable-e2e if not set up yet.

---

**If user selects Parallel Tests:**

```
Parallel Tests — test-writer agent runs alongside builder during /orbit:build
Current: [enabled/disabled]

[1] Enable  [2] Disable  [3] Back
```

If enabling: set parallel_tests.enabled = true in config.md.
If disabling: set parallel_tests.enabled = false.

Note: requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1.

---

**If user selects Agent Teams:**

```
Agent Teams — parallel research (observe) and review (integrate)
Current: [enabled/disabled]

[1] Enable  [2] Disable  [3] Back
```

Set agent_teams.enabled = true/false in config.md.

Note: requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1.

**Step 3: Write config**

Create or update `.orbit/config.md`:

```markdown
# Project Config

**Project:** [project_name]
**Updated:** [timestamp]

## Project Settings

```yaml
project:
  name: [project_name]
  version: [version or "0.0.0"]
```

## Integrations

### SonarQube

```yaml
sonarqube:
  enabled: [true/false]
  project_key: [key]
```

### E2E Testing

```yaml
e2e:
  enabled: false
  runner: playwright-cli
  browser: chromium
  default: false
  base_url: ""
```

### Parallel Tests

```yaml
parallel_tests:
  enabled: false   # requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

## Preferences

```yaml
preferences:
  auto_commit: false
  verbose_output: false
```

---
*Config updated: [timestamp]*
```

**Step 4: Confirm**

```
════════════════════════════════════════
CONFIG UPDATED
════════════════════════════════════════

Integrations:
  SonarQube: [enabled/disabled]

Config saved to: .orbit/config.md

────────────────────────────────────────
[If SonarQube just enabled:]
▶ NEXT: /orbit:quality-gate
  Run your first code quality scan.

[Otherwise:]
Configuration complete.
────────────────────────────────────────
```

</process>

<output>
- `.orbit/config.md` created or updated
- Integration status changed as requested
- Clear next steps if applicable
</output>
