---
name: orbti:refine-fix
description: Create a fix refine (REFINE variant) from UAT issues
argument-hint: "<refine, e.g., '04-02'>"
allowed-tools: [Read, Bash, Write, Glob, Grep, AskUserQuestion]
---

<model>sonnet</model>

<objective>
Create FIX.md refine from UAT issues found during verify.

**When to use:** After `/orbti:test` logs issues to project-scoped UAT file.

**Output:** `{refine}-FIX.md` in the project directory, ready for execution.
</objective>

<execution_context>
@~/.claude/orbti-framework/references/refine-format.md
@~/.claude/orbti-framework/references/checkpoints.md
</execution_context>

<context>
Refine number: $ARGUMENTS (required - e.g., "04-02" or "10-01")

@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>

<step name="parse">
**Parse refine argument:**

$ARGUMENTS should be a refine number like "04-02" or "10-01".
Extract project number (XX) and refine number (NN).

If no argument provided:
```
Error: Refine number required.

Usage: /orbti:refine-fix 04-02

This creates a fix refine from .orbti/projects/XX-name/{refine}-UAT.md
```
Exit.
</step>

<step name="find">
**Find UAT.md file:**

Search for matching UAT file:
```bash
ls .orbti/projects/*/{refine}-UAT.md 2>/dev/null
```

If not found:
```
No UAT.md found for refine {refine}.

UAT.md files are created by /orbti:test when testing finds issues.
If no issues were found during testing, no fix refine is needed.
```
Exit.
</step>

<step name="read">
**Read issues:**

Read the UAT.md file.
Parse each issue:
- ID (UAT-NNN)
- Title
- Severity (Blocker/Major/Minor/Cosmetic)
- Description/steps to reproduce
- AC reference

Count total issues by severity.
</step>

<step name="refine">
**Create fix tasks:**

For each issue (or logical group):
- Create one task per issue OR
- Group related minor issues into single task

Task structure:
```xml
<task type="auto">
  <name>Fix UAT-001: [issue title]</name>
  <files>[affected files from issue]</files>
  <action>
[What to fix based on issue description]
[Reference original acceptance criteria]
  </action>
  <verify>[Test that issue is resolved]</verify>
  <done>[Issue acceptance criteria met]</done>
</task>
```

Prioritize: Blocker → Major → Minor → Cosmetic
</step>

<step name="write">
**Write FIX.md:**

Create `.orbti/projects/XX-name/{refine}-FIX.md`:

```markdown
---
project: XX-name
refine: {refine}-FIX
type: fix
wave: 1
depends_on: []
files_modified: [files from issues]
autonomous: true
---

<objective>
## Goal
Fix {N} UAT issues from refine {refine}.

## Purpose
Address issues discovered during user acceptance testing.

## Output
All issues resolved, ready for re-verification.

Source: {refine}-UAT.md
Priority: {blocker count} blocker, {major count} major, {minor count} minor, {cosmetic count} cosmetic
</objective>

<context>
@.orbti/STATE.md
@.orbti/ROADMAP.md

**Issues being fixed:**
@.orbti/projects/XX-name/{refine}-UAT.md

**Original refine for reference:**
@.orbti/projects/XX-name/{refine}-REFINE.md
</context>

<acceptance_criteria>
[Generate AC from issues - each issue becomes an AC]
</acceptance_criteria>

<tasks>
[Generated fix tasks]
</tasks>

<boundaries>
## DO NOT CHANGE
- Files not related to the issues
- Core functionality that passed testing

## SCOPE LIMITS
- Only fix issues from {refine}-UAT.md
- No scope creep or additional improvements
</boundaries>

<verification>
Before declaring refine complete:
- [ ] All blocker issues fixed
- [ ] All major issues fixed
- [ ] Minor/cosmetic issues fixed or documented as deferred
- [ ] Original acceptance criteria from issues met
</verification>

<success_criteria>
- All UAT issues from {refine}-UAT.md addressed
- Ready for re-verification with /orbti:test
</success_criteria>

<output>
After completion, create `.orbti/projects/XX-name/{refine}-FIX-INTEGRATE.md`
</output>
```
</step>

<step name="offer">
**Offer execution:**

```
════════════════════════════════════════
FIX REFINE CREATED
════════════════════════════════════════

{refine}-FIX.md — {N} issues to fix

| Severity | Count |
|----------|-------|
| Blocker  | {n}   |
| Major    | {n}   |
| Minor    | {n}   |
| Cosmetic | {n}   |

────────────────────────────────────────
Continue to BUILD?

[1] Approved, run BUILD | [2] Review first | [3] Pause here
────────────────────────────────────────
```

Use AskUserQuestion to get response.
If approved: `/orbti:build .orbti/projects/XX-name/{refine}-FIX.md`
</step>

</process>

<success_criteria>
- [ ] UAT.md found and parsed
- [ ] Fix tasks created for each issue (or grouped)
- [ ] FIX.md written with proper ORBTI structure
- [ ] User offered to execute or review
</success_criteria>
