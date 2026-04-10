---
name: orbti:research-phase
description: Research technical unknowns before planning using subagents
argument-hint: "[topic or question]"
allowed-tools: [Read, Task, Bash, Write]
---

<model>sonnet</model>

<objective>
Identify and research technical unknowns before planning. Runs before /orbti:refine — no project exists yet.

**When to use:** Before planning when there are specific technical questions to investigate.

**Distinction from /orbti:cocreate:**
- `/orbti:research-phase`: Gathers information, does not decide
- `/orbti:cocreate`: Researches to make a decision

**Subagent orchestration:** Spawns multiple research agents in parallel for independent unknowns.
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/research.md
@./.claude/orbti-framework/references/subagent-criteria.md
</execution_context>

<context>
Topic: $ARGUMENTS (optional)

@.orbti/PROJECT.md
@.orbti/STATE.md
</context>

<process>

<step name="identify_unknowns" priority="first">
**Identify what to research:**

If $ARGUMENTS provided, use as starting point.
Otherwise, ask: "What do you want to investigate before planning?"

Present identified unknowns for confirmation before spawning agents.
</step>

<step name="analyze_unknowns">
Analyze the phase for research needs:

1. Read ROADMAP.md phase description
2. Read PROJECT.md for context
3. Check for existing CONTEXT.md in phase directory

**Identify unknowns in these categories:**

| Category | Examples |
|----------|----------|
| **Codebase** | "How does X work in this project?", patterns to follow |
| **Web/Docs** | Library features, API patterns, best practices |
| **Comparison** | "Should we use X or Y?", tradeoff analysis |
| **Architecture** | Design patterns, integration approaches |

**Classification logic:**
- Codebase unknowns → Explore agent
- Web/docs unknowns → general-purpose agent

**Filter for substantial unknowns:**
- Only include items that need 15+ min research
- Trivial lookups handled in main session
- Max 3 parallel research agents (token efficiency)

**Present findings:**
```
════════════════════════════════════════
RESEARCH UNKNOWNS
════════════════════════════════════════

Topic: {topic}

Identified unknowns:
1. {unknown_1} [{codebase|web}]
2. {unknown_2} [{codebase|web}]
3. {unknown_3} [{codebase|web}]

{If more than 3:}
Additional unknowns (will research after initial batch):
4. {unknown_4}
5. {unknown_5}

────────────────────────────────────────
Proceed with research? [yes/no/modify list]
────────────────────────────────────────
```

Wait for user confirmation.
</step>

<step name="spawn_research_agents">
**Subagent criteria validation (from subagent-criteria.md):**

| Criterion | Check |
|-----------|-------|
| Task Independence | Each unknown is self-contained ✓ |
| Clear Scope | Unknown defines research question ✓ |
| Parallel Value | Multiple agents run simultaneously ✓ |
| Complexity Sweet Spot | Each 15-30 min expected ✓ |
| Token Efficiency | Batch spawn, consolidated return ✓ |
| State Compatibility | No blocking checkpoints ✓ |

**Spawn agents in parallel:**

Use multiple Task tool calls in single message:

```
Task 1 (Codebase unknowns):
- subagent_type: Explore
- description: "Research: {unknown}"
- run_in_background: true (if multiple)
- prompt: [research prompt]

Task 2 (Web unknowns):
- subagent_type: general-purpose
- description: "Research: {unknown}"
- run_in_background: true (if multiple)
- prompt: [research prompt]
```

Display:
```
Spawning {N} research agents...

Agent 1: {unknown_1} (Explore)
Agent 2: {unknown_2} (general-purpose)
Agent 3: {unknown_3} (Explore)

Researching in parallel. This may take a few minutes...
```
</step>

<step name="consolidate_findings">
When all agents complete:

1. Create context research directory:
   ```bash
   mkdir -p .orbti/context/research
   ```

2. Save individual findings:
   - `.orbti/context/research/{unknown-slug}.md`

3. Create consolidated RESEARCH.md:
   - `.orbti/context/RESEARCH.md`
   - Summarizes all findings
   - Links to individual research files

4. Present summary:
```
════════════════════════════════════════
RESEARCH COMPLETE
════════════════════════════════════════

Topic: {topic}
Unknowns researched: {count}

Summary:
1. {unknown_1}: {key finding}
2. {unknown_2}: {key finding}
3. {unknown_3}: {key finding}

Detailed findings:
- .orbti/context/RESEARCH.md (consolidated)
- .orbti/context/research/*.md (individual)

────────────────────────────────────────
This research informs but does not automatically integrate into refines.

What's next?
[1] Review consolidated findings
[2] Plan this (/orbti:refine)
[3] Discuss goals first (/orbti:observe)
[4] Done for now
────────────────────────────────────────
```
</step>

</process>

<success_criteria>
- [ ] Unknowns identified and classified
- [ ] User confirmed research list
- [ ] Appropriate agents spawned in parallel
- [ ] Findings consolidated into phase RESEARCH.md
- [ ] Summary presented for review
</success_criteria>
