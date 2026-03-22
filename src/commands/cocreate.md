---
name: orbti:cocreate
description: Research technical options and make decisions before planning a phase
argument-hint: "<project or topic>"
allowed-tools: [Read, Bash, Glob, Grep, WebSearch, WebFetch, Task, AskUserQuestion]
---

<objective>
Execute technical discovery to inform planning decisions. Produces COCREATE.md with findings, recommendation, and confidence level.

**When to use:** Before planning a project with technical unknowns (library selection, architecture decisions, integration approaches).

**Distinct from /orbti:research:** Research gathers documentation/information. Cocreate makes technical decisions.

**Not part of the main loop** — run explicitly when there are genuine technical unknowns to resolve.
</objective>

<model>opus</model>

<execution_context>
@~/.claude/orbti-framework/workflows/cocreate.md
@~/.claude/orbti-framework/templates/COCREATE.md
@~/.claude/orbti-framework/references/model-routing.md
</execution_context>

<context>
$ARGUMENTS (project number or topic)

@.orbti/STATE.md
@.orbti/ROADMAP.md
</context>

<process>
**Follow workflow: @~/.claude/orbti-framework/workflows/cocreate.md**

The workflow implements:
1. Determine depth level (quick/standard/deep)
2. Identify unknowns for the project
3. Research options using subagents
4. Cross-verify findings
5. Create COCREATE.md with recommendation
6. Assign confidence level
7. Route to planning when complete
</process>

<success_criteria>
- [ ] Cocreate depth determined
- [ ] Unknowns identified
- [ ] Options researched with sources
- [ ] COCREATE.md created (for standard/deep)
- [ ] Recommendation provided with confidence
- [ ] Ready for /orbti:refine
</success_criteria>
