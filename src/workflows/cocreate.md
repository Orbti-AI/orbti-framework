<purpose>
Execute discovery at appropriate depth to inform planning decisions. Produces COCREATE.md with findings, recommendation, and confidence level.

Distinct from research workflow: Research gathers information. Discovery makes technical decisions.
</purpose>

<when_to_use>
- Before planning a phase with technical unknowns
- When choosing between libraries/frameworks
- For architectural decisions
- When integrating new external services
</when_to_use>

<depth_levels>

| Level | Name | Time | Output | When |
|-------|------|------|--------|------|
| 1 | Quick | 2-5 min | Verbal confirmation | Confirming known approach |
| 2 | Standard | 15-30 min | COCREATE.md | Choosing between options |
| 3 | Deep | 1+ hour | Detailed COCREATE.md | Novel problems, high-risk |

</depth_levels>

<process>

<step name="determine_depth">
**Determine discovery depth:**

If $ARGUMENTS specifies depth (e.g., "deep auth options"):
- Use specified depth

Otherwise, infer from context:
- Single known library, just verifying → Level 1 (quick)
- Comparing 2-3 options → Level 2 (standard)
- Novel problem, architectural decision → Level 3 (deep)

Confirm with user if unclear:
```
Discovery for: [topic]
Suggested depth: [level] - [rationale]

Proceed? (yes / suggest different depth)
```
</step>

<step name="level_1_quick">
**Level 1: Quick Verification (2-5 minutes)**

For: Confirming known approach still valid.

1. Quick search for current state of library/approach
2. Verify:
   - Still actively maintained
   - API unchanged from expectations
   - No critical issues

3. **If verified:** Proceed to planning. No COCREATE.md needed.
4. **If concerns:** Escalate to Level 2.

Output: Verbal confirmation to proceed.
</step>

<step name="level_2_standard">
**Level 2: Standard Discovery (15-30 minutes)**

For: Choosing between options, new integration.

**Check if agent teams are enabled:**
```bash
grep "agent_teams:" .orbti/config.md 2>/dev/null | grep "enabled: true"
```

Teams are **off by default** — only active if `agent_teams.enabled: true` in `.orbti/config.md`.
Enable via `/orbti:config` → Agent Teams → Enable.

**If teams enabled: spawn research team**

Create a team with 2-3 researcher teammates, each assigned a different option or perspective. Teammates challenge each other's findings before producing COCREATE.md.

```
Create a research team for: [topic]
- researcher-A: investigate option A — document pros, cons, real-world usage
- researcher-B: investigate option B — same criteria
- researcher-C (if 3+ options): investigate option C
Instructions: after individual research, challenge each other's findings.
Reach a consensus recommendation.
```

Team lead synthesizes debate output into COCREATE.md.

**If teams not available: use sequential subagents**

1. **Identify what to discover:**
   - What options exist?
   - Key comparison criteria?
   - Our specific use case?

2. **Research each option** via Task subagents (parallel):
   - Official documentation
   - Current version/status
   - Key features for our use case

3. **Compare options:**
   - Build comparison table
   - Note pros/cons for each
   - Identify deal-breakers

4. **Reference quality patterns:**
   @src/references/research-quality-control.md

**Both paths produce:**
5. **Create COCREATE.md:**
   Use template: @src/templates/COCREATE.md
   - Summary with recommendation
   - Findings per option
   - Confidence level (should be MEDIUM-HIGH)

Output: `.orbti/projects/XX-name/COCREATE.md`
</step>

<step name="level_3_deep">
**Level 3: Deep Dive (1+ hour)**

For: Architectural decisions, novel problems.

**Check if agent teams are active:**
```bash
echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}"
grep "agent_teams:" .orbti/config.md 2>/dev/null | grep "enabled: false"
```

Teams are **on by default** when env var is set — only skip if `agent_teams.enabled: false` in config.

**If teams available (value = 1): spawn adversarial debate team**

Spawn 3-5 researcher teammates in explicit debate mode — each one's job is not only to investigate their theory but to actively challenge the others'.

```
Create an adversarial research team for: [topic]
Spawn [N] teammates, each assigned a different hypothesis or approach.
Instructions:
- Each teammate researches their approach independently first
- Then teammates actively try to disprove each other's findings
- Flag contradictions and unresolved disagreements
- Produce a consensus COCREATE.md only after the debate
- Document minority opinions if no clear consensus
```

Team lead monitors debate, resolves deadlocks, synthesizes final COCREATE.md.

**Confidence gate for teams:**
If teammates cannot reach consensus:
```
Research team could not reach consensus on: [issue]

Disagreement: [summary of competing views]

Options:
1. Dig deeper (extend the debate)
2. Proceed with majority view (document dissent)
3. Pause (need human input on direction)
```

**If teams not available: use sequential subagents**

1. **Scope the discovery:**
   - Define clear scope
   - List specific questions to answer
   - Set include/exclude boundaries

2. **Exhaustive research** via subagents:
   - All relevant libraries/frameworks
   - Architecture patterns
   - Best practices / known limitations
   - Production experiences and gotchas

3. **Cross-verify ALL findings:**
   - Every claim verified with authoritative source
   - Mark verified vs assumed
   - Flag contradictions

4. **Confidence gate:**
   If LOW confidence on critical finding:
   ```
   Discovery confidence is LOW: [reason]

   Options:
   1. Dig deeper (more research)
   2. Proceed anyway (accept uncertainty)
   3. Pause (need to think)
   ```

**Both paths produce:**
5. **Create comprehensive COCREATE.md:**
   - Full template structure
   - Quality report with sources
   - Confidence per finding
   - Validation checkpoints if LOW confidence

Output: Comprehensive `.orbti/projects/XX-name/COCREATE.md`
</step>

<step name="route_to_planning">
**Complete discovery and route:**

```
════════════════════════════════════════
DISCOVERY COMPLETE
════════════════════════════════════════

Topic: [what was discovered]
Depth: Level [N]
Confidence: [HIGH/MEDIUM/LOW]

Recommendation: [one-liner]

Output: [path to COCREATE.md or "verbal confirmation"]

────────────────────────────────────────
▶ NEXT: /orbti:refine [phase]
────────────────────────────────────────
```
</step>

</process>

<success_criteria>
**Level 1:**
- [ ] Known approach verified
- [ ] Verbal confirmation to proceed

**Level 2:**
- [ ] Options researched
- [ ] Comparison made
- [ ] COCREATE.md created
- [ ] Confidence MEDIUM+

**Level 3:**
- [ ] Scope defined
- [ ] Exhaustive research
- [ ] All findings cross-verified
- [ ] COCREATE.md created
- [ ] Confidence gate passed
</success_criteria>
