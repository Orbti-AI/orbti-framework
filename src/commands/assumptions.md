---
name: orbit:assumptions
description: Surface Claude's assumptions about a phase before planning
argument-hint: "<phase-number>"
allowed-tools: [Read, Bash]
---

<model>opus</model>

<objective>
Surface Claude's assumptions about a phase to validate understanding before planning.

**When to use:** Before planning to catch misconceptions early.

**Distinction from /orbit:cocreate:** This command shows what CLAUDE thinks. The discuss command gathers what USER wants.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/phase-assumptions.md
</execution_context>

<context>
Phase number: $ARGUMENTS (required)

@.orbit/PROJECT.md
@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
Follow workflow: @~/.claude/orbit-framework/workflows/phase-assumptions.md
</process>

<success_criteria>
- [ ] Assumptions presented across 5 areas
- [ ] Confidence levels indicated
- [ ] User can provide corrections
- [ ] Clear path to planning after validation
</success_criteria>
