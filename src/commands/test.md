---
name: orbit:test
description: Run integration tests against acceptance criteria — auto-detects test runner, writes missing tests, then falls back to manual UAT
argument-hint: "[optional: phase or plan number, e.g., '4' or '04-02'] [--manual] [--e2e]"
allowed-tools: [Read, Bash, Glob, Grep, Edit, Write, AskUserQuestion, Task]
---

<model>sonnet</model>

<objective>
Validate that what was built satisfies the acceptance criteria defined in LOOP.md.

**Primary mode:** Auto-detect the project's test runner, write integration tests for any ACs without coverage, run them, and map results back to AC-1, AC-2, etc.

**Fallback:** If no test runner is detected and --e2e is not passed, fall back to guided manual UAT.

**Flags:**
- `--manual` — skip auto-detection, go straight to manual UAT
- `--e2e` — attempt Playwright CLI for browser-based tests (requires `npx playwright` available)
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/test-auto.md
@~/.claude/orbit-framework/workflows/verify-work.md
@~/.claude/orbit-framework/templates/TEST.md
</execution_context>

<context>
Scope: $ARGUMENTS (optional)
- If provided: Test specific phase or plan (e.g., "4" or "04-02")
- If not provided: Test most recently completed plan

@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
**If --manual flag present:**
  Follow workflow: @~/.claude/orbit-framework/workflows/verify-work.md

**Otherwise:**
  Follow workflow: @~/.claude/orbit-framework/workflows/test-auto.md
</process>

<success_criteria>
- [ ] Test scope identified (from LOOP.md or INTEGRATE.md)
- [ ] ACs extracted for validation
- [ ] Test runner detected (or manual mode triggered)
- [ ] Integration tests written for uncovered ACs
- [ ] Tests executed and results captured
- [ ] Results mapped to AC-1, AC-2... (PASS/FAIL)
- [ ] Issues logged to `.orbit/projects/XX-name/{plan}-UAT.md`
- [ ] Summary presented with verdict
- [ ] User knows next steps
</success_criteria>

<anti_patterns>
- Don't install new test frameworks — use what the project already has
- Don't skip writing tests — if an AC has no test, write one before running
- Don't fix issues during testing — capture for /orbit:refine-fix
- Don't assume pass — run the tests, read the output
- Don't use Playwright unless --e2e flag is passed or it's already installed
</anti_patterns>
