---
name: orbti:test
description: Run integration tests against acceptance criteria — auto-detects test runner, auto-runs Playwright when configured, writes missing tests, then falls back to manual UAT
argument-hint: "[optional: phase or refine number, e.g., '4' or '04-02'] [--manual] [--e2e]"
allowed-tools: [Read, Bash, Glob, Grep, Edit, Write, AskUserQuestion, Task]
---

<model>sonnet</model>

<objective>
Validate that what was built satisfies the acceptance criteria defined in REFINE.md.

**Primary mode:** Auto-detect the project's test runner, write integration tests for any ACs without coverage, run them, and map results back to AC-1, AC-2, etc.

**E2E mode (auto):** If `e2e.enabled: true` in `config.md` OR `playwright-cli` declared as required in `SPECIAL-FLOWS.md` AND Playwright is installed → E2E runs automatically for ALL ACs. No flag needed.

**Fallback:** If no test runner is detected and no E2E configured, fall back to guided manual UAT.

**Flags:**
- `--manual` — skip auto-detection, go straight to manual UAT
- `--e2e` — force Playwright E2E regardless of config (requires `playwright-cli` or `npx playwright` available)
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/test-auto.md
@./.claude/orbti-framework/workflows/verify-work.md
@./.claude/orbti-framework/templates/TEST.md
</execution_context>

<context>
Scope: $ARGUMENTS (optional)
- If provided: Test specific phase or refine (e.g., "4" or "04-02")
- If not provided: Test most recently completed refine

@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>
**If --manual flag present:**
  Follow workflow: @./.claude/orbti-framework/workflows/verify-work.md

**Otherwise:**
  Follow workflow: @./.claude/orbti-framework/workflows/test-auto.md
</process>

<success_criteria>
- [ ] Test scope identified (from REFINE.md or INTEGRATE.md)
- [ ] ACs extracted for validation
- [ ] Test runner detected (or manual mode triggered)
- [ ] Integration tests written for uncovered ACs
- [ ] Tests executed and results captured
- [ ] Results mapped to AC-1, AC-2... (PASS/FAIL)
- [ ] Issues logged to `.orbti/projects/XX-name/{refine}-UAT.md`
- [ ] Summary presented with verdict
- [ ] User knows next steps
</success_criteria>

<anti_patterns>
- Don't install new test frameworks — use what the project already has
- Don't skip writing tests — if an AC has no test, write one before running
- Don't fix issues during testing — capture for /orbti:refine-fix
- Don't assume pass — run the tests, read the output
- Don't use Playwright unless `e2e.enabled: true` in config, playwright declared in SPECIAL-FLOWS, or `--e2e` flag passed
</anti_patterns>
