# REFINE.md Template

Template for `.orbti/projects/{name}/{refine}-REFINE.md` - executable project refines.

**Naming:** `{refine}-REFINE.md` (e.g., `02-REFINE.md` for Refine 2 of any project)

---

## File Template

```markdown
---
project: name
refine: NN
type: execute                    # execute | tdd | research
wave: N                          # Execution wave (1, 2, 3...). Pre-computed at refine time.
depends_on: []                   # Refine IDs this refine requires (e.g., ["01-01"]).
files_modified: []               # Files this refine modifies.
autonomous: true                 # false if refine has checkpoints requiring user interaction
---

<objective>
## Goal
[What this refine accomplishes - specific, measurable]

## Purpose
[Why this matters for the project - connects to PROJECT.md value]

## Output
[What artifacts will be created/modified]
</objective>

<context>
## Project Context
@.orbti/PROJECT.md
@.orbti/ROADMAP.md
@.orbti/STATE.md

## Prior Work (only if genuinely needed)
# Only reference prior SUMMARYs if:
# - This refine imports types/exports from prior refine
# - Prior refine made decision affecting this refine
# - Prior refine's output is direct input to this refine
#
# Do NOT reflexively chain: Refine 02 refs 01, Refine 03 refs 02...

## Source Files
@path/to/relevant/source.ts
</context>

<skills>
## Required Skills (from SPECIAL-FLOWS.md)

<!-- If .orbti/SPECIAL-FLOWS.md exists, this section is auto-populated by /orbti:refine -->
<!-- If no SPECIAL-FLOWS.md, omit this section or show "No specialized flows configured" -->

| Skill | Priority | When to Invoke | Loaded? |
|-------|----------|----------------|---------|
| /skill-name | required | Before [work type] | ○ |
| /skill-name | optional | For [specific need] | ○ |

**BLOCKING:** Required skills MUST be loaded before BUILD proceeds.
Run each skill command or confirm already loaded.

## Skill Invocation Checklist
- [ ] /skill-1 loaded (run command or confirm)
- [ ] /skill-2 loaded (run command or confirm)

</skills>

<acceptance_criteria>

## AC-1: [Criterion Name]
```gherkin
Given [precondition / system state]
When [user action / trigger]
Then [expected outcome / observable result]
```

## AC-2: [Criterion Name]
```gherkin
Given [precondition]
When [action]
Then [outcome]
```

## AC-3: [Criterion Name]
```gherkin
Given [precondition]
When [action]
Then [outcome]
```

</acceptance_criteria>

<tasks>

<task type="auto">
  <name>Task 1: [Action-oriented name]</name>
  <files>path/to/file.ext, another/file.ext</files>
  <action>
    [Specific implementation instructions]
    - What to do
    - How to do it
    - What to avoid and WHY
  </action>
  <verify>[Command or check to prove it worked]</verify>
  <done>[Measurable acceptance criteria - links to AC-N]</done>
</task>

<task type="auto">
  <name>Task 2: [Action-oriented name]</name>
  <files>path/to/file.ext</files>
  <action>[Specific implementation]</action>
  <verify>[Command or check]</verify>
  <done>[Acceptance criteria]</done>
</task>

<task type="checkpoint:decision" gate="blocking">
  <decision>[What needs deciding]</decision>
  <context>[Why this decision matters now]</context>
  <options>
    <option id="option-a">
      <name>[Option name]</name>
      <pros>[Benefits and advantages]</pros>
      <cons>[Tradeoffs and limitations]</cons>
    </option>
    <option id="option-b">
      <name>[Option name]</name>
      <pros>[Benefits and advantages]</pros>
      <cons>[Tradeoffs and limitations]</cons>
    </option>
  </options>
  <resume-signal>Select: option-a or option-b</resume-signal>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[What was built that needs verification]</what-built>
  <how-to-verify>
    1. Run: [command to start]
    2. Visit: [URL or location]
    3. Test: [Specific interactions]
    4. Confirm: [Expected behaviors]
  </how-to-verify>
  <resume-signal>Type "approved" to continue, or describe issues to fix</resume-signal>
</task>

</tasks>

<boundaries>

## DO NOT CHANGE
- [Protected file or pattern]
- [Another protected element]

## SCOPE LIMITS
- [What's explicitly out of scope for this refine]
- [Another exclusion]

</boundaries>

<verification>
Before declaring refine complete:
- [ ] [Specific test command]
- [ ] [Build/type check passes]
- [ ] [Behavior verification]
- [ ] All acceptance criteria met
</verification>

<success_criteria>
- All tasks completed
- All verification checks pass
- No errors or warnings introduced
- [Refine-specific criteria]
</success_criteria>

<output>
After completion, create `.orbti/projects/{name}/{refine}-INTEGRATE.md`
</output>
```

---

## Frontmatter Fields

| Field | Required | Purpose |
|-------|----------|---------|
| `project` | Yes | Project identifier (e.g., `foundation`) |
| `refine` | Yes | Refine number within project (e.g., `01`, `02`) |
| `type` | Yes | `execute` for standard, `tdd` for test-driven, `research` for exploration |
| `wave` | Yes | Execution wave number (1, 2, 3...). Pre-computed at refine time. |
| `depends_on` | Yes | Array of refine IDs this refine requires. Empty = parallel candidate. |
| `files_modified` | Yes | Files this refine touches. For conflict detection. |
| `autonomous` | Yes | `true` if no checkpoints, `false` if has checkpoints |

---

## Task Types

| Type | Use For | Behavior |
|------|---------|----------|
| `auto` | Everything AI can do independently | Fully autonomous execution |
| `checkpoint:decision` | Implementation choices requiring human input | Pauses, presents options, waits for selection |
| `checkpoint:human-verify` | Visual/functional verification | Pauses, presents verification steps, waits for approval |
| `checkpoint:human-action` | Truly unavoidable manual steps (rare) | Pauses, describes action, waits for confirmation |

---

## Task Structure Requirements

Every `auto` task MUST have:
- `<name>` - Action-oriented, describes outcome
- `<files>` - Which files created/modified
- `<action>` - Specific implementation (what to do, what to avoid)
- `<verify>` - How to prove it worked (command, check)
- `<done>` - Acceptance criteria (links to AC-N)

**If you can't specify Files + Action + Verify + Done, the task is too vague.**

---

## Acceptance Criteria Format

Use Given/When/Then (BDD) format:

```gherkin
Given [precondition / initial state]
When [action / trigger]
Then [expected outcome]
```

**Guidelines:**
- Each criterion should be independently testable
- Include error states and edge cases
- Avoid implementation details (describe behavior, not code)
- Link tasks to criteria via `<done>AC-N satisfied</done>`

---

## Scope Guidance

**Refine sizing:**
- 2-3 tasks per refine maximum
- ~50% context usage target
- Single concern per refine

**When to split into multiple refines:**
- Different subsystems (auth vs API vs UI)
- More than 3 tasks
- Risk of context overflow
- TDD candidates (separate refines)

**Prefer vertical slices:**
```
PREFER: Refine 01 = User (model + API + UI)
        Refine 02 = Product (model + API + UI)

AVOID:  Refine 01 = All models
        Refine 02 = All APIs
        Refine 03 = All UIs
```

---

## Parallel Execution

**Wave 1 candidates (parallel):**
- `depends_on: []` - No dependencies
- No file conflicts with other Wave 1 refines
- `autonomous: true` - No checkpoints blocking

**Sequential (genuine dependency):**
- `depends_on: ["01-01"]` - Requires prior loop output
- Uses types/exports created by prior loop
- Prior loop makes decision affecting this loop

**Do NOT reflexively chain:** Refine 02 depending on 01 "just because" creates false sequential execution.

---

## Anti-Patterns

**Vague tasks:**
```xml
<!-- BAD -->
<task type="auto">
  <name>Set up authentication</name>
  <action>Add auth to the app</action>
</task>

<!-- GOOD -->
<task type="auto">
  <name>Create login endpoint with JWT</name>
  <files>src/api/auth/login.ts, src/lib/jwt.ts</files>
  <action>
    Create POST /api/auth/login endpoint:
    - Accept { email, password } body
    - Validate against User model with bcrypt compare
    - Return JWT token on success (15min expiry)
    - Return 401 on invalid credentials
    Avoid: Storing plaintext passwords, using weak JWT secrets
  </action>
  <verify>curl -X POST /api/auth/login returns JWT token</verify>
  <done>AC-1 satisfied: User can authenticate with valid credentials</done>
</task>
```

**Reflexive dependencies:**
```yaml
# BAD - chaining just because sequential
depends_on: ["01-01"]  # Loop 02 doesn't actually need 01's output

# GOOD - genuine dependency
depends_on: ["01-01"]  # Loop 02 imports User type from 01-01
```

**Missing boundaries:**
```markdown
<!-- BAD - no protection -->
<tasks>...</tasks>

<!-- GOOD - explicit boundaries -->
<boundaries>
## DO NOT CHANGE
- database/migrations/* (schema locked for this phase)
- src/lib/auth.ts (auth system stable)
</boundaries>
```
