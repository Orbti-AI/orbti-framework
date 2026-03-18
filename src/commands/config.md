---
name: orbit:config
description: View and manage ORBIT project configuration and integrations
allowed-tools: [Read, Write, Edit, Bash, AskUserQuestion]
---

<model>haiku</model>

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

[3] Test Writer - Write integration tests alongside /orbit:build (default: disabled)
    Status: [enabled/disabled]
    Note: if Agent Teams active, tests written in parallel; otherwise written sequentially

[4] Agent Teams - Parallel research and review (default: disabled)
    Status: [enabled/disabled]

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

If enabling:

1. Check if Playwright CLI binary is installed:
   ```bash
   playwright-cli --version 2>/dev/null || echo "NOT_INSTALLED"
   ```

2. Check if the Playwright CLI skill is installed:
   ```bash
   ls ~/.claude/skills/playwright-cli 2>/dev/null || echo "NOT_INSTALLED"
   ```

3. If either is missing — show instructions and **do NOT enable**:
   ```
   ════════════════════════════════════════
   Playwright CLI not ready
   ════════════════════════════════════════

   Missing: [playwright-cli binary / skill / both]

   To install:
     npm install -g @playwright/cli@latest
     playwright-cli install --skills
     playwright-cli install chromium

   Run /orbit:config again after installing.
   ════════════════════════════════════════
   ```

4. If both installed: set `e2e.enabled: true` in config.md and confirm:
   ```
   ✓ Playwright CLI E2E enabled
     binary: installed
     skill: installed
   ```

If disabling: set `e2e.enabled: false` in config.md.

---

**If user selects Test Writer:**

```
Test Writer — writes integration tests for each AC during /orbit:build
Current: [enabled/disabled]

When enabled:
  • Agent Teams active → test-writer agent runs in parallel with builder
  • Agent Teams inactive → tests written sequentially after each task

[1] Enable  [2] Disable  [3] Back
```

If enabling: set test_writer.enabled = true in config.md.
If disabling: set test_writer.enabled = false.

---

**If user selects Agent Teams:**

```
Agent Teams — parallel research (observe) and code review (integrate)
Current: [enabled/disabled]
Default: disabled

[1] Enable  [2] Disable  [3] Back
```

**If enabling:**

1. Set `agent_teams.enabled: true` in `.orbit/config.md`
2. Write `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` to `.claude/settings.json`:
   ```bash
   cat .claude/settings.json 2>/dev/null || echo "{}"
   ```
   Merge into `env` block. Create file if it doesn't exist.
3. Confirm:
   ```
   ✓ Agent Teams enabled
     .orbit/config.md → agent_teams.enabled: true
     .claude/settings.json → CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
   Restart Claude Code to activate.
   ```

**If disabling:**
1. Set `agent_teams.enabled: false` in `.orbit/config.md`
2. Remove `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` from `.claude/settings.json`

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

### Test Writer

```yaml
test_writer:
  enabled: false   # parallel if Agent Teams active, sequential otherwise
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
