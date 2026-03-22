<purpose>
Auto-detect the project's test runner, write integration tests for any acceptance criteria without coverage, run tests, and map results to ACs.

Falls back to manual UAT if no test runner is found and --e2e is not passed.
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

<step name="identify_scope">
**Determine what to test:**

If $ARGUMENTS provided: find corresponding REFINE.md and INTEGRATE.md
If not: use most recently modified REFINE.md

Read REFINE.md to extract:
- All acceptance criteria (AC-1, AC-2, ...)
- Files created/modified (from INTEGRATE.md if exists)
- Boundaries (what was NOT changed)
</step>

<step name="detect_runner">
**Auto-detect test runner:**

Run the detection commands above.

**E2E trigger — check any of these sources (in order):**
```bash
# 1. config.md explicit flag
grep -A3 "^e2e:" .orbti/config.md 2>/dev/null | grep "enabled: true"

# 2. SPECIAL-FLOWS declaration of playwright as required
grep -i "playwright" .orbti/SPECIAL-FLOWS.md 2>/dev/null | grep -i "required"
```

**Playwright availability — check both variants:**
```bash
playwright-cli --version 2>/dev/null || \
npx playwright --version 2>/dev/null || \
echo "NOT_INSTALLED"
```

Use whichever variant is available. Prefer `playwright-cli` if both present.

If E2E triggered by any source above AND playwright available → run `handle_e2e` step.
If E2E triggered but playwright not available → warn, continue with integration tests only.

If `--e2e` flag passed manually → always attempt E2E regardless of config.

If no runner found and no E2E source:
```
No test runner detected in this project.

Falling back to manual UAT.
```
→ Switch to: @~/.claude/orbti-framework/workflows/verify-work.md
</step>

<step name="map_acs_to_tests">
**Verify tests were written during BUILD:**

Tests should already exist — they are written during BUILD alongside the implementation.

For each AC in the REFINE.md, confirm a test exists:
```bash
grep -r "AC-[0-9]" tests/ spec/ __tests__/ test/ 2>/dev/null
```

If any AC has no test:
- Warn: "AC-N has no test — was BUILD completed? Writing test now."
- Write the missing test before running (same rules as BUILD: one test per AC, behavior not implementation, no new dependencies)
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

<step name="handle_e2e">
**E2E with Playwright — auto-runs when configured, covers ALL ACs.**

Check config first:
```bash
grep -A3 "^e2e:" .orbti/config.md 2>/dev/null
npx playwright --version 2>/dev/null || echo "NOT_INSTALLED"
```

**Decision matrix:**

| `e2e.enabled` | Playwright available | `--e2e` flag | Action |
|--------------|---------------------|--------------|--------|
| `true` | yes | any | **Auto-run E2E for ALL ACs** |
| `true` | no | any | Warn + skip E2E, continue to human verification |
| `false` | any | `--e2e` | Run E2E for ALL ACs (flag override) |
| `false` | any | absent | Skip E2E |

**If E2E should run:**

1. Read `base_url` from `.orbti/config.md` (e2e.base_url).
   If empty, ask: "What is your app's base URL? (e.g. http://localhost:3000)"

2. Verify app is reachable:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" <base_url>
   ```
   If not reachable → present `checkpoint:human-action`:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Start the application
   ════════════════════════════════════════
   The app must be running at <base_url> for E2E tests.
   Start it (e.g. npm run dev), then type "ready".
   ════════════════════════════════════════
   ```

3. **Check for login automation in SPECIAL-FLOWS:**
   ```bash
   grep -A5 "playwright" .orbti/SPECIAL-FLOWS.md 2>/dev/null | grep -i "login\|auth\|credential"
   ```
   If login config found → include login steps in each test (navigate to login URL, fill credentials, submit).
   Credentials come from SPECIAL-FLOWS config — never hardcode in test files; read from env or config.

4. For **each AC** in REFINE.md (not just user-facing ones), write a Playwright test:
   ```bash
   # Tests go in tests/e2e/ or playwright/ (match project convention)
   # Inspect page structure first using available CLI:
   playwright-cli goto <base_url> 2>/dev/null || npx playwright open <base_url>
   playwright-cli snapshot 2>/dev/null
   ```
   Rules per AC:
   - One test file per AC, named `ac-N-<slug>.spec.ts`
   - Test behavior observable via the browser, not internals
   - No new npm dependencies beyond `@playwright/test`
   - Take screenshot at end of each test as evidence: `ac-N-<slug>.png`

5. Run all E2E tests:
   ```bash
   # If playwright-cli available:
   playwright-cli test tests/e2e/ 2>/dev/null || \
   npx playwright test tests/e2e/ --reporter=list
   ```

6. Map results to ACs:
   ```
   AC-1: [description] ........... PASS (screenshot: ac-1.png)
   AC-2: [description] ........... FAIL — [error]
   AC-3: [description] ........... PASS (screenshot: ac-3.png)
   ```

7. E2E failures are **warnings**, not blockers:
   - Log each failure to `.orbti/projects/{name}/{refine}-UAT.md`
   - Do not block INTEGRATE for E2E failures
   - Integration test failures block; E2E failures warn
</step>

<step name="report_and_route">
**Present results and Playwright checkpoint:**

```
════════════════════════════════════════
TEST RESULTS: [Refine Name]
════════════════════════════════════════

Runner: [detected runner]
Tests written: [N new] / [M existing]

AC-1: ✓ PASS
AC-2: ✗ FAIL — [error]
AC-3: ✓ PASS

Tests: [total] | Passed: [N] | Failed: [N]

════════════════════════════════════════
Verdict: [ALL PASS | FAILURES FOUND]
════════════════════════════════════════
```

**If FAILURES FOUND:**
- Log each failing AC to `.orbti/projects/{name}/{refine}-UAT.md`
- Offer: "Run /orbti:refine-fix to address failing tests before integrating"
- User can override and continue anyway (with issues logged)

**Playwright checkpoint — always show after test results:**

```
════════════════════════════════════════
CHECKPOINT: E2E com Playwright
════════════════════════════════════════

Quer ativar o agente Playwright para rodar os
testes E2E cobrindo todos os ACs?

O agente vai:
1. Navegar até a app em execução
2. Executar um cenário por AC
3. Capturar screenshot de evidência
4. Mapear: AC-1 ✓ / AC-2 ✗

[1] Sim, rodar Playwright agora
[2] Não, ir direto para INTEGRATE
════════════════════════════════════════
```

**If "1" / "sim" / "yes" / "playwright":**
→ Follow `handle_e2e` step — agent runs Playwright for ALL ACs.
After E2E completes, offer INTEGRATE:
```
Continue to INTEGRATE? [1] Yes | [2] Review issues first
```

**If "2" / "não" / "skip":**
→ Offer INTEGRATE directly:
```
Continue to INTEGRATE? [1] Yes | [2] Pause here
```

Accept "1", "yes", "go", "continue" → run `/orbti:integrate [refine-path]`

```
────────────────────────────────────────
⚠ Testes automatizados não substituem a verificação humana.
────────────────────────────────────────
```

→ Follow: @~/.claude/orbti-framework/workflows/verify-work.md

Human verification runs for every AC, independent of automated results. An AC that passes automated tests can still fail human review — that outcome is valid and must be captured.
</step>

</process>

<success_criteria>
- [ ] Test runner detected (or manual fallback triggered)
- [ ] ACs extracted from REFINE.md
- [ ] Existing test coverage mapped
- [ ] Missing tests written (one per uncovered AC)
- [ ] Tests executed
- [ ] Results mapped to AC-1, AC-2...
- [ ] Issues logged if any failures
- [ ] Human verification checklist presented (always — automated tests do not substitute)
- [ ] Human verdict captured for each AC
- [ ] User routed to integrate or refine-fix
</success_criteria>
