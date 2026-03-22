<refine_format>

## Purpose

REFINE.md IS the executable prompt. It contains everything needed to execute a project: objective, context, acceptance criteria, tasks, boundaries, verification, and output specification.

**Core principle:** A refine is Claude-executable when Claude can read the REFINE.md and immediately start implementing without asking clarifying questions.

If Claude has to guess, interpret, or make assumptions - the task is too vague.

## Frontmatter

Every REFINE.md starts with YAML frontmatter:

```yaml
---
project: name
refine: NN
type: execute
wave: N
depends_on: []
files_modified: []
autonomous: true
---
```

| Field | Required | Purpose |
|-------|----------|---------|
| `project` | Yes | Project identifier (e.g., `rules-layer`) |
| `refine` | Yes | Refine number within project (e.g., `01`, `02`) |
| `type` | Yes | `execute` for standard, `tdd` for test-driven, `research` for exploration |
| `wave` | Yes | Execution wave number (pre-computed at refine time) |
| `depends_on` | Yes | Array of refine IDs this refine requires |
| `files_modified` | Yes | Files this refine touches (for conflict detection) |
| `autonomous` | Yes | `true` if no checkpoints, `false` if has checkpoints |

## Loop Structure

```markdown
---
[frontmatter]
---

<objective>
## Goal
[What this refine accomplishes - specific, measurable]

## Purpose
[Why this matters for the project]

## Output
[What artifacts will be created/modified]
</objective>

<context>
@.orbti/PROJECT.md
@.orbti/ROADMAP.md
@.orbti/STATE.md
@relevant/source/files.ts
</context>

<acceptance_criteria>
## AC-1: [Criterion Name]
Given [precondition]
When [action]
Then [expected outcome]
</acceptance_criteria>

<tasks>
[Task definitions]
</tasks>

<boundaries>
## DO NOT CHANGE
[Protected files/patterns]

## SCOPE LIMITS
[What's explicitly out of scope]
</boundaries>

<verification>
[Overall phase checks]
</verification>

<success_criteria>
[Measurable completion criteria]
</success_criteria>

<output>
[INTEGRATE.md specification]
</output>
```

## Task Anatomy

Every `auto` task has four required fields:

### files
**What it is:** Exact file paths created or modified.

```xml
<!-- GOOD -->
<files>src/app/api/auth/login/route.ts, prisma/schema.prisma</files>

<!-- BAD -->
<files>the auth files, relevant components</files>
```

### action
**What it is:** Specific implementation instructions, including what to avoid and WHY.

```xml
<!-- GOOD -->
<action>
  Create POST endpoint accepting {email, password}.
  Query User by email, compare password with bcrypt.
  On match, create JWT with jose library (15-min expiry).
  Return 200. On mismatch, return 401.
  Avoid: jsonwebtoken (CommonJS issues with Edge runtime)
</action>

<!-- BAD -->
<action>Add authentication</action>
```

### verify
**What it is:** How to prove the task is complete.

```xml
<!-- GOOD -->
<verify>curl -X POST localhost:3000/api/auth/login returns 200 with Set-Cookie header</verify>

<!-- BAD -->
<verify>It works</verify>
```

### done
**What it is:** Acceptance criteria - links to AC-N for traceability.

```xml
<!-- GOOD -->
<done>AC-1 satisfied: Valid credentials return 200 + JWT cookie</done>

<!-- BAD -->
<done>Authentication is complete</done>
```

**If you can't specify Files + Action + Verify + Done, the task is too vague.**

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

## Boundaries Section

ORBTI refines include explicit boundaries:

```markdown
<boundaries>
## DO NOT CHANGE
- database/migrations/* (schema locked for this phase)
- src/lib/auth.ts (auth system stable)

## SCOPE LIMITS
- This refine creates API only - no UI
- Do not add new dependencies
</boundaries>
```

Boundaries prevent scope creep by making off-limits areas explicit.

## Specificity Levels

### Too Vague
```xml
<task type="auto">
  <name>Add authentication</name>
  <files>???</files>
  <action>Implement auth</action>
  <verify>???</verify>
  <done>Users can authenticate</done>
</task>
```
Claude: "How? What type? What library? Where?"

### Just Right
```xml
<task type="auto">
  <name>Create login endpoint with JWT</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    POST endpoint accepting {email, password}.
    Query User by email, compare password with bcrypt.
    On match, create JWT with jose (15-min expiry).
    Return 200. On mismatch, return 401.
  </action>
  <verify>curl -X POST returns 200 with Set-Cookie header</verify>
  <done>AC-1 satisfied: Valid credentials → 200 + cookie</done>
</task>
```
Claude can implement immediately.

### Too Detailed
Writing the actual code in the refine. Trust Claude to implement from clear instructions.

## Sizing Guidance

**Good refine size:** 2-3 tasks, ~50% context usage, single concern.

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
```

## Anti-Patterns

**Vague actions:**
- "Set up the infrastructure"
- "Handle edge cases"
- "Make it production-ready"

**Unverifiable completion:**
- "It works correctly"
- "User experience is good"
- "Code is clean"

**Missing context:**
- "Use the standard approach"
- "Follow best practices"
- "Like the other endpoints"

**Reflexive dependencies:**
```yaml
# BAD - chaining just because sequential
depends_on: ["01-01"]  # Loop 02 doesn't actually need 01's output

# GOOD - genuine dependency
depends_on: ["01-01"]  # Loop 02 imports User type from 01-01
```

</refine_format>
