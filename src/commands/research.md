---
name: orbti:research
description: Research a topic using subagents for discovery
argument-hint: "<topic> [--project <slug>] [--codebase | --web]"
allowed-tools: [Read, Task, Bash, Write]
---

<model>sonnet</model>

<objective>
Research a specific topic using subagents and save findings for review.

**Dois modos de saída:**
- **Standalone** (sem projeto): salva em `.orbti/research/{topic-slug}.md`
- **Project-aware** (com projeto): adiciona seção ao `.orbti/projects/{slug}/RESEARCH.md`

**Quando usar:** Antes de planejar, quando há perguntas técnicas ou estratégicas a investigar.
O research informa — não auto-integra.

**Subagent use case:** Research is the APPROPRIATE use of subagents per subagent-criteria.md.
</objective>

<execution_context>
@./.claude/orbti-framework/workflows/research.md
@./.claude/orbti-framework/references/subagent-criteria.md
</execution_context>

<context>
Topic: $ARGUMENTS (required)

Optional flags:
- `--project <slug>`: Salva no RESEARCH.md do projeto (cria se não existe)
- `--codebase`: Focus on codebase exploration (uses Explore agent)
- `--web`: Focus on web/documentation (uses general-purpose agent)
- No flag: Auto-detect based on topic

@.orbti/PROJECT.md
@.orbti/STATE.md
</context>

<process>
Follow workflow: @./.claude/orbti-framework/workflows/research.md
</process>

<success_criteria>
- [ ] Topic validated (not trivial)
- [ ] Appropriate agent type selected
- [ ] Subagent spawned for research
- [ ] **Se --project:** findings adicionados ao .orbti/projects/{slug}/RESEARCH.md
- [ ] **Se standalone:** findings salvos em .orbti/research/{topic-slug}.md
- [ ] Summary presented for review
</success_criteria>
