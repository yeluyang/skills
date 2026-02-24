# Step 7: Execution Plan

Convert the approved spec into an execution plan. Write it to a file.

**Path:** `docs/plans/YYYY-MM-DD-refactoring-plan.md`

## Task Decomposition

Break the spec into executable tasks — one task per chunk (or per sub-chunk if a chunk is too large for a single unit of work).

For each task:

```markdown
### Task: <descriptive name>

**Chunk:** <spec chunk name>
**Blocked by:** <task IDs, or "none">
**Files touched:** <list of files this task will create / modify / delete>

**Changes:**

- <specifics of what to do>

**Verification:** <test command, type check, or other validation>
```

## Dependency & File-Overlap Analysis

After decomposing tasks, build two graphs:

1. **Dependency graph** — which tasks must complete before others can start (from chunk dependencies)
2. **File-overlap graph** — which tasks touch the same files (from the "Files touched" field)

A task group is **safe to parallelize** only when tasks within it have **no dependency edges AND no file overlap**.

## Execution Strategy Recommendation

Based on the analysis, recommend one of:

- **Sequential (single agent)** — when most tasks share files or have dense dependencies. Default for safety; the agent sees full state at every step and can course-correct.
- **Parallel (agent team)** _(recommended when eligible)_ — when the analysis produces multiple task groups with zero file overlap between groups. Parallel execution is faster and equally safe when isolation is proven.

Present the recommendation and rationale to the user. Let the user choose.

## Dependency Graph Output

Summarize the structure in the plan file:

```markdown
## Execution Groups

**Recommended strategy:** <sequential / parallel>
**Rationale:** <why — e.g., "3 independent groups with zero file overlap" or "heavy file overlap across chunks">

- Group 1 (no blockers): [Task A, Task D, Task G]
- Group 2 (blocked by Group 1): [Task B, Task E]
- Group 3 (blocked by Group 2): [Task C, Task F]
```

Commit the plan document.
