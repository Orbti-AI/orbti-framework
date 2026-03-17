---
name: orbit:observe
description: Research technical options and make decisions before planning a phase
argument-hint: "<project or topic>"
allowed-tools: [Read, Bash, Glob, Grep, WebSearch, WebFetch, Task, AskUserQuestion]
---

<objective>
Execute technical discovery to inform planning decisions. Produces OBSERVE.md with findings, recommendation, and confidence level.

**When to use:** Before planning a project with technical unknowns (library selection, architecture decisions, integration approaches).

**Distinct from /orbit:research:** Research gathers documentation/information. Observe makes technical decisions.

**Not part of the main loop** — run explicitly when there are genuine technical unknowns to resolve.
</objective>

<execution_context>
@~/.claude/orbit-framework/workflows/observe.md
@~/.claude/orbit-framework/templates/OBSERVE.md
</execution_context>

<context>
$ARGUMENTS (project number or topic)

@.orbit/STATE.md
@.orbit/ROADMAP.md
</context>

<process>
**Follow workflow: @~/.claude/orbit-framework/workflows/observe.md**

The workflow implements:
1. Determine depth level (quick/standard/deep)
2. Identify unknowns for the project
3. Research options using subagents
4. Cross-verify findings
5. Create OBSERVE.md with recommendation
6. Assign confidence level
7. Route to planning when complete
</process>

<success_criteria>
- [ ] Observe depth determined
- [ ] Unknowns identified
- [ ] Options researched with sources
- [ ] OBSERVE.md created (for standard/deep)
- [ ] Recommendation provided with confidence
- [ ] Ready for /orbit:refine
</success_criteria>
