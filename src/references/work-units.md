<work_units>

<purpose>
Guide refine sizing to maintain consistent quality. Refines must complete within ~50% context to avoid quality degradation. This reference helps estimate scope and know when to split.

**For CARL users:** CARL context brackets handle this operationally. This reference documents the underlying principles.
</purpose>

<quality_insight>
Claude degrades when it perceives context pressure and enters "completion mode."

| Context Usage | Quality | Claude's State |
|---------------|---------|----------------|
| 0-30% | PEAK | Thorough, comprehensive |
| 30-50% | GOOD | Confident, solid work |
| 50-70% | DEGRADING | Efficiency mode begins |
| 70%+ | POOR | Rushed, minimal |

**The 40-50% inflection point:** Claude sees context mounting and thinks "I'd better conserve now." Result: quality crash.

**The rule:** Stop BEFORE quality degrades, not at context limit.
</quality_insight>

<context_target>
**Refines should complete within ~50% of context usage.**

Why 50% not 80%?
- No context anxiety possible
- Quality maintained start to finish
- Room for unexpected complexity
- If you target 80%, you've already spent 40% in degradation mode
</context_target>

<task_limits>
**Each refine: 2-3 tasks maximum. Stay under 50% context.**

| Task Complexity | Tasks/Refine | Context/Task | Total |
|-----------------|------------|--------------|-------|
| Simple (config, single file) | 3 | ~10-15% | ~30-45% |
| Medium (feature, few files) | 2 | ~20-30% | ~40-50% |
| Complex (architecture, many files) | 1-2 | ~30-40% | ~30-50% |

**When in doubt: Default to 2 tasks.** Better to have an extra refine than degraded quality.
</task_limits>

<split_signals>

**Always split when:**
- More than 3 tasks (even if tasks seem small)
- Multiple subsystems (DB + API + UI = separate refines)
- Any task with >5 file modifications
- Discovery + implementation (COCREATE.md in one refine, implementation in another)

**Consider splitting when:**
- Estimated >5 files modified total
- Complex domains (auth, payments, data modeling)
- Uncertainty about approach
- Natural semantic boundaries

</split_signals>

<splitting_strategies>

**Vertical slices (default):** Group by feature, not by layer.

```
PREFER: Refine 01 = User (model + API + UI)
        Refine 02 = Product (model + API + UI)

AVOID:  Refine 01 = All models
        Refine 02 = All APIs (depends on 01)
```

Vertical slices maximize parallelism and reduce dependencies.

**By dependency:** Only when genuine dependencies exist.
```
Refine 01: Auth foundation (middleware, JWT)
Refine 02: Protected features (uses auth from 01)
```

**By complexity:** When one slice is much heavier.
```
Refine 01: Dashboard layout shell
Refine 02: Data fetching and state
Refine 03: Visualization components
```
</splitting_strategies>

<estimating_context>

| Files Modified | Context Impact |
|----------------|----------------|
| 0-3 files | ~10-15% (small) |
| 4-6 files | ~20-30% (medium) |
| 7+ files | ~40%+ (large - split) |

| Complexity | Context/Task |
|------------|--------------|
| Simple CRUD | ~15% |
| Business logic | ~25% |
| Complex algorithms | ~40% |
| Domain modeling | ~35% |

**Quick estimates:**
- 2 simple tasks: ~30%
- 2 medium tasks: ~50%
- 2 complex tasks: ~80% (split!)
- 3 simple tasks: ~45%
- 3 medium tasks: ~75% (risky)
</estimating_context>

<orbti_integration>
**How this fits ORBTI's loop:**

```
REFINE ──▶ BUILD ──▶ INTEGRATE
```

- **REFINE phase:** Use work-units to size correctly
- **BUILD phase:** Execute within context budget
- **INTEGRATE phase:** Record actual duration for future estimates

**With CARL (recommended):**
CARL context brackets automate these checks:
- FRESH (>60%): Full implementation OK
- MODERATE (40-60%): Consider splitting
- DEPLETED (<40%): Handoff or fresh agent

ORBTI provides the framework. CARL provides operational automation.
</orbti_integration>

<anti_patterns>

**Bad - Comprehensive refine:**
```
Refine: "Complete Authentication System"
Tasks: 8
Result: Task 1-3 good, Task 4-5 degrading, Task 6-8 rushed
```

**Good - Atomic refines:**
```
Refine 1: "Auth Database Models" (2 tasks)
Refine 2: "Auth API Core" (2 tasks)
Refine 3: "Auth UI Components" (2 tasks)
Each: 30-40% context, peak quality
```
</anti_patterns>

<summary>
**Core rules:**
- 2-3 tasks per refine
- 50% context target
- Split when in doubt
- Vertical slices over horizontal layers

**The principle:** Aggressive atomicity. More refines, smaller scope, consistent quality.
</summary>

</work_units>
