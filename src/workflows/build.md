<purpose>
Execute an approved REFINE.md by running tasks in order, verifying each, handling checkpoints, and recording results for INTEGRATE phase reconciliation.
</purpose>

<when_to_use>
- User has approved a REFINE.md (explicit approval required)
- STATE.md shows loop position at REFINE complete, ready for BUILD
- No unresolved blockers from planning project
</when_to_use>

<loop_context>
Expected phase: BUILD
Prior phase: REFINE (approval just received)
Next phase:  INTEGRATE (after execution completes)
</loop_context>

<required_reading>
@.orbti/STATE.md
@.orbti/projects/{project}/{refine}-REFINE.md
</required_reading>

<references>
@~/.claude/orbti-framework/references/checkpoints.md (if refine has checkpoints)
@~/.claude/orbti-framework/references/loop-phases.md
</references>

<process>

<step name="check_test_writer" priority="blocking">
**BLOCKING CHECK: Must run before any task execution. Do not skip.**

Check if Test Writer is enabled:

```bash
grep "test_writer:" .orbti/config.md 2>/dev/null | grep "enabled: true"
grep "agent_teams:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Both off by default. Enable via `/orbti:config`.

| Test Writer | Agent Teams | BUILD behavior |
|-------------|-------------|----------------|
| OFF (default) | OFF | Build only — no test writing |
| OFF (default) | ON  | Build only — no test writing. Agent Teams active in other phases (cocreate, integrate) |
| ON | OFF | Sequential — write test after each task completes |
| ON | ON  | Parallel — spawn builder + test-writer agents simultaneously |

Route:
- `test_writer.enabled: false` (any teams state) → proceed to `execute_tasks`, no test writing
- `test_writer.enabled: true` AND teams ON → skip to `parallel_team_build`
- `test_writer.enabled: true` AND teams OFF → proceed to `execute_tasks` with sequential test writing

If this step was skipped and tasks were already executed: log deviation to STATE.md Decisions:
`| [date]: check_test_writer skipped — test_writer config not checked before BUILD | Project [N] | Tests may need to be written manually post-build |`
</step>

<step name="validate_approval" priority="first">
1. Confirm user has explicitly approved the refine
   - Do NOT assume approval
   - Look for explicit signal: "approved", "execute", "go ahead", "background", etc.
2. Read STATE.md to verify:
   - Loop position shows REFINE complete
   - Correct phase and refine identified
3. If approval unclear:
   - Ask: "Refine ready at [path]. Approve refine execution?"
   - Wait for explicit approval before proceeding
4. Check execution mode:
   - If invoked via `/orbti:build-bg` → route to `background_build` step
   - Otherwise → proceed with foreground execution
</step>

<step name="background_build">
**Background mode — only available for refines with `autonomous: true`.**

Check REFINE.md frontmatter:
```
autonomous: true   → background allowed
autonomous: false  → has checkpoints, cannot run unattended
```

If `autonomous: false`:
```
This refine has checkpoints that require your input — it cannot run in the background.
Run in foreground instead? [yes / no]
```

If `autonomous: true`, spawn a background agent:

```
Spawn background agent for: [refine-path]

The agent must:
1. Execute all tasks in order (same rules as execute_tasks step)
2. Run each <verify> step
3. Respect all <boundaries>
4. On task failure: log the failure, stop, do NOT continue to next task
5. On completion: update STATE.md loop position to BUILD ✓
6. Report summary when done

The agent does NOT pause for checkpoints — refine must be fully autonomous.
```

Confirm to user:
```
════════════════════════════════════════
BUILD RUNNING IN BACKGROUND
════════════════════════════════════════

Refine: [refine-path]
Tasks: [N] tasks

You will be notified when complete.
You can continue other work in the meantime.
════════════════════════════════════════
```

When background agent completes, present the same BUILD COMPLETE summary
as the foreground finalize step, then offer INTEGRATE.
</step>

<step name="load_plan">
1. Read the REFINE.md file
2. Parse frontmatter:
   - autonomous: determines checkpoint handling
   - files_modified: track what we'll change
   - depends_on: verify dependencies met
3. Extract tasks from <tasks> section
4. Note boundaries from <boundaries> section
5. Load acceptance criteria for verification reference
</step>

<step name="verify_required_skills" priority="blocking">
**BLOCKING CHECK: Required skills must be loaded before execution.**

1. Check if REFINE.md has a <skills> section
2. If no <skills> section: proceed (no skill requirements)
3. If <skills> section exists:
   a. For each skill marked "required":
      - Check if skill has been invoked in current session
      - If not invoked: add to missing_skills list
   b. If missing_skills is not empty:
      - **BLOCK execution**
      - Display:
        ```
        ════════════════════════════════════════
        ⛔ BLOCKED: Required skills not loaded
        ════════════════════════════════════════

        This refine requires the following skills:

        Missing:
        - /skill-name → Run: /skill-name
        - /skill-name → Run: /skill-name

        Load these skills now, then type "ready" to continue.
        Or type "override" to proceed without (not recommended).
        ════════════════════════════════════════
        ```
      - Wait for user input
      - If "ready": re-check skills, proceed if all loaded
      - If "override":
        - Log deviation to STATE.md Decisions: "Override: Proceeded without required skills [list]"
        - Proceed with warning
   c. If all required skills loaded:
      - Display: "✓ All required skills loaded"
      - Proceed to execute_tasks

**This check runs BEFORE any task execution, ensuring skills are in place.**
</step>

<step name="execute_tasks">
For each <task> in order:

**Solution checkpoint — before each task:**

Read `Solution Intent` from CONTEXT.md (or REFINE.md objective). State explicitly:
```
WHO: [who this serves]
WHAT: [what they accomplish with this task's output]
FEEL: [how the solution should behave — fast? explicit? forgiving?]
```

This prevents defaulting to the most common implementation pattern. Every decision in the task — data model, error handling, API shape, or UI component — should connect back to these answers.

If CONTEXT.md has no Solution Intent: proceed, but note the absence. Defaults are more likely.

**Skip the solution checkpoint when the task has no interface** — purely internal work (architecture, security, infra, refactor with no interaction surface). The REFINE.md ACs define success for these tasks.

**If the task produces visual UI:**

Visual craft is delegated to the agentic-design skill declared in `.orbti/SPECIAL-FLOWS.md`.

If agentic-design was loaded (confirmed by `verify_required_skills`):
- The skill handles design system, tokens, and craft review
- Feed it the Solution Intent from CONTEXT.md as WHO/WHAT/FEEL input when invoking

If agentic-design is NOT declared in SPECIAL-FLOWS.md:
```
This task builds UI. Visual craft is not enforced without the agentic-design skill.
→ Add it to .orbti/SPECIAL-FLOWS.md as required and reload, OR
→ Type "proceed" to continue — Solution Intent will guide decisions but no craft system
```
If user proceeds without the skill: apply Solution Intent (WHO/WHAT/FEEL) to visual decisions at minimum.

**Agent learning (all task types):**
After task execution, if ANY of these occurred:
- User corrected an approach or redirected the implementation
- A tool or command behaved unexpectedly
- A deviation from REFINE was required
- A technique worked unusually well

→ Append to `.orbti/RUNBOOK.md` under the relevant category:
```
1. **[YYYY-MM-DD] [Short rule title]**
   Do instead: [concrete repeatable action]
```
Only log recurring, high-value guidance. Skip one-off events.

**If type="auto":**
1. Log task start: "Task N: [name]"
2. Execute <action> content:
   - Create/modify files specified in <files>
   - Follow specific instructions
   - Respect boundaries (DO NOT CHANGE protected files)
3. Run <verify> command/check
4. Record result:
   - PASS: verification succeeded
   - FAIL: verification failed (stop and report)
5. Note <done> criteria satisfied
6. **If sequential test writing active** (`test_writer.enabled: true` AND teams not active):
   Write integration test for the AC this task satisfies (from <done> field).
   Use project's existing test runner and conventions. One test per AC, behavior not implementation.

**If type="checkpoint:human-verify":**
1. Stop execution
2. Present checkpoint clearly:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Human Verification
   ════════════════════════════════════════

   Task [N] of [Total]: [name]

   What was built:
   [what-built content]

   How to verify:
   [how-to-verify content]

   [resume-signal content]
   ════════════════════════════════════════
   ```
3. Wait for user response
4. On "approved": continue to next task
5. On issues reported: address issues, re-verify, or note deviation

**If type="checkpoint:decision":**
1. Stop execution
2. Present decision with options:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Decision Required
   ════════════════════════════════════════

   Decision: [decision content]
   Context: [context content]

   Options:
   [option-a]: [name] - Pros: [pros] / Cons: [cons]
   [option-b]: [name] - Pros: [pros] / Cons: [cons]

   [resume-signal content]
   ════════════════════════════════════════
   ```
3. Wait for user selection
4. **Record decision to STATE.md:**
   - Open `.orbti/STATE.md`
   - Find `### Decisions` under `## Accumulated Context`
   - Add row: `| [date]: [Decision summary] | Project [N] | [Impact on work] |`
   - Example: `| 2026-01-28: Install in sandbox for testing | Project 1 | Project created in sandbox/box2-orbti-test |`
5. Continue with chosen direction

**If type="checkpoint:human-action":**
1. Stop execution
2. Present required action:
   ```
   ════════════════════════════════════════
   CHECKPOINT: Human Action Required
   ════════════════════════════════════════

   Action: [action content]

   Instructions:
   [instructions content]

   After completion, I will verify:
   [verification content]

   [resume-signal content]
   ════════════════════════════════════════
   ```
3. Wait for user confirmation
4. Run verification check
5. Continue if verified, report if failed
</step>

<step name="parallel_team_build">
**Team mode: builder + test-writer in parallel**

Only runs when `test_writer.enabled: true` AND `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

Spawn a 2-agent team sharing the same task list:

```
Create a build team for refine: [refine-path]

Spawn 2 teammates:

- builder: implement the tasks in REFINE.md in order.
  For each task: execute <action>, run <verify>, log result.
  After each task completes, post to shared task list: "task-N done: [AC satisfied]"

- test-writer: write integration tests for each AC as builder completes tasks.
  Watch the task list for "task-N done" signals.
  For each completed task: write a focused integration test for the AC it satisfies.
  Rules:
    - Use the project's existing test runner (detect from package.json / pytest.ini / go.mod)
    - Place tests in the project's existing test directory (match conventions)
    - One test per AC, named clearly referencing the AC
    - Test behavior, not implementation details
    - No new dependencies
  Post to task list when each test is written: "test-AC-N written"

Both agents respect REFINE.md boundaries (DO NOT CHANGE sections).
Builder blocks on checkpoints — test-writer continues working on other ACs.
```

Team lead monitors both agents. On builder checkpoint: pause team, present checkpoint to user, resume after response.

After both agents complete:
- Verify all tasks have a corresponding test
- Any missing: test-writer fills the gap before proceeding
- Proceed to `finalize` step
</step>

<step name="handle_failures">
If a task verification fails:

1. **Stop immediately** - do not proceed to next task
2. **Report clearly:**
   - Which task failed
   - What verification check failed
   - What was expected vs actual
3. **Offer options:**
   - Retry: attempt the task again
   - Skip: mark as failed, continue (creates deviation)
   - Stop: halt execution, prepare for debugging
4. **Record resolution to STATE.md:**
   - Add to `### Decisions` section: `| [date]: Task [N] [retry/skip/stop] - [reason] | Project [N] | [impact] |`
</step>

<step name="track_progress">
Throughout execution:

1. Maintain mental log of:
   - Tasks completed (with results)
   - Tasks failed (with reasons)
   - Checkpoints resolved (with decisions/approvals)
   - Deviations from refine
2. This information feeds into INTEGRATE phase
</step>

<step name="finalize">
After all tasks attempted:

1. Summarize execution:
   - Tasks completed: N of M
   - Failures: list any
   - Deviations: list any

2. **Run tests** (if `test_writer.enabled: true` or tests exist for this refine):
   - Run the project's test command scoped to the files/ACs touched in this build
   - Collect: total tests, passed, failed, duration

3. Update STATE.md:
   - Loop position: REFINE ✓ → BUILD ✓ → INTEGRATE ○
   - Last activity: timestamp and completion status

5. Report build results:
   ```
   ════════════════════════════════════════
   BUILD COMPLETE
   ════════════════════════════════════════
   [execution summary]

   Tests written: [N tests for ACs: AC-1, AC-2, AC-3 | none — test_writer off]
   ────────────────────────────────────────
   ```

6. **Route to TEST:**
   ```
   ---
   Continue to TEST?

   [1] Yes, run /orbit:test | [2] Pause here
   ```
   **Accept quick inputs:** "1", "yes", "test", "go" → run `/orbit:test [refine-path]`

   TEST handles: running the written tests, E2E checkpoint, and routing to INTEGRATE.
</step>

</process>

<output>
- Modified files as specified in REFINE.md
- Execution log (mental, for INTEGRATE)
- STATE.md updated with BUILD complete
</output>

<error_handling>
**Refine not found:**
- Check STATE.md for correct path
- Ask user to confirm refine location

**Boundary violation attempted:**
- Stop immediately
- Report which boundary would be violated
- Do not modify protected files

**Verification command fails:**
- Report the failure
- Offer retry/skip/stop options
- Do not mark task as complete

**Checkpoint timeout:**
- Remind user checkpoint is waiting
- Do not proceed without response
- Offer to save state and continue later
</error_handling>

<anti_patterns>
**Assuming approval:**
Do NOT start BUILD without explicit user approval of the refine.

**Skipping check_test_writer:**
Do NOT jump directly to task execution without checking test_writer config first. With `test_writer: true` + `agent_teams: true`, the correct mode is parallel team build — skipping this step silently drops the test-writing behavior. Always run `check_test_writer` before any task. If already skipped, log the deviation and write tests manually post-build.

**REFINE.md created before test_writer feature existed:**
If the REFINE refine was approved in a prior ORBTI version that lacked `check_test_writer`, the REFINE.md will not reference this step — the build executes as if test writing doesn't exist, even with `test_writer: true` in config. Detection: config has test_writer enabled but no tests were written during build.
Mitigation: After completing build, check config → if `test_writer: true`, write tests manually for each AC before INTEGRATE. Log to STATE.md: `| [date]: REFINE.md pre-dates test_writer feature — tests written manually post-build | Project [N] | All ACs covered |`

**Skipping verification:**
Every task MUST have its verify step run. No "it looks right" assumptions.

**Ignoring boundaries:**
If a task would modify a protected file, STOP. Do not rationalize violations.

**Proceeding past checkpoints:**
Checkpoints are blocking. Do not continue without user response.

**Creating a new REFINE after BUILD without INTEGRATE:**
After BUILD completes, the ONLY valid next steps are TEST or INTEGRATE.
Do NOT create a new REFINE.md while INTEGRATE ○ is open.
If the user asks to create another refine after BUILD, respond:
"Loop not closed — run /orbti:integrate [refine-path] first."
No exceptions.
</anti_patterns>
