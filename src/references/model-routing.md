<overview>
ORBIT uses different Claude models based on task complexity and cost profile. The goal: use the most capable model where judgment matters, and the fastest/cheapest where it doesn't.

Available models (as of Claude 4.x):
- `claude-opus-4-6` — Opus: deepest reasoning, highest cost, slowest
- `claude-sonnet-4-6` — Sonnet: balanced capability/cost, default workhorse
- `claude-haiku-4-5-20251001` — Haiku: fastest, cheapest, best for lightweight tasks
</overview>

<routing_table>

## Default Model per Phase/Command

| Phase / Command        | Model   | Reason |
|------------------------|---------|--------|
| `/orbit:cocreate`      | Opus    | Vision articulation, creative synthesis |
| `/orbit:observe`       | Opus    | Architecture decisions, trade-off analysis |
| `/orbit:refine`        | Opus    | Planning quality directly affects all downstream work |
| `/orbit:research`      | Opus    | Synthesis of complex, ambiguous information |
| `/orbit:assumptions`   | Opus    | Surfacing blind spots requires deep reasoning |
| `/orbit:build`         | Sonnet  | Execution of an approved plan — capability over cost |
| `/orbit:test`          | Sonnet  | Test writing + result analysis |
| `/orbit:integrate`     | Sonnet  | Reconciliation and deviation analysis |
| `/orbit:refine-fix`      | Sonnet  | Debugging and fix planning |
| `/orbit:debug`         | Sonnet  | Root cause analysis |
| `/orbit:progress`      | Haiku   | Status read — no reasoning required |
| `/orbit:pause`         | Haiku   | Handoff file generation — structured, low-complexity |
| `/orbit:resume`        | Haiku   | Context restore from file |
| `/orbit:config`        | Haiku   | Config read/write — purely mechanical |
| `/orbit:help`          | Haiku   | Static reference lookup |
| Subagents (research)   | Sonnet  | Parallelized research tasks — Opus overkill at scale |
| Subagents (background) | Sonnet  | Build agents spawned by `/orbit:build-bg` |

</routing_table>

<override>

## Overriding Model in Config

Projects can override the default model for any phase in `.orbit/config.md`:

```yaml
models:
  default: sonnet           # Fallback for any phase not listed
  overrides:
    refine: opus            # Keep Opus for planning (recommended)
    build: haiku            # Override if project is simple/fast
    observe: sonnet         # Override if cost is a concern
```

Valid values: `opus`, `sonnet`, `haiku`

If no override is set, defaults from the routing table above apply.

</override>

<when_to_use>

## Guidance for Choosing

**Use Opus when:**
- The output directly shapes what gets built (REFINE, OBSERVE, COCREATE)
- The task involves ambiguity that requires judgment
- A wrong answer is expensive to fix downstream

**Use Sonnet when:**
- The plan is clear and execution is the primary challenge
- Writing code, tests, or structured documents
- Analysis with defined scope (INTEGRATE, DEBUG)

**Use Haiku when:**
- Reading files and summarizing state (progress, status)
- Generating structured output from a clear template
- Any task where a wrong answer can be quickly corrected

</when_to_use>

<subagent_usage>
When spawning subagents via the Agent tool, pass the `model` parameter:

```
Agent(model: "opus")   → complex research/planning subagents
Agent(model: "sonnet") → build/test/integrate subagents
Agent(model: "haiku")  → status/lookup subagents
```

For parallel research agents (e.g., in `/orbit:research`): use `sonnet` by default.
Opus at scale multiplies cost without proportional quality gain.
</subagent_usage>
