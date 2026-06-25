# Step 8: Execute

Execute using whichever strategy the user approved in Step 7.

## Test Discipline During Refactoring

Applies to both execution modes. Refactoring inevitably touches tests — modifications, additions, deletions. Two non-negotiable rules:

1. **Preserve the spirit of existing edge-case tests.** Existing tests — especially those covering boundary conditions, race conditions, error paths, and corner cases — are often hard-won knowledge (production incidents, subtle bug discoveries). When adapting a test to fit refactored code, the scenario it describes and the defensive thinking behind it must survive intact. Changing the code under test is fine; losing the edge case is not. If a test can no longer be expressed against the new structure, flag it to the user — do not silently delete it.

2. **Audit new code for new edge cases.** Refactoring changes structure, and new structure can introduce new boundary conditions (new interface contracts, new error propagation paths, new state transitions). After implementing each task, ask: does this change create edge cases that didn't exist before? If yes, add tests for them.

## Sequential Mode (single agent)

Execute tasks in dependency order. After each task:

1. Run tests to verify no regressions
2. If the change reveals issues not anticipated in the plan, pause and surface them to the user before continuing

## Parallel Mode (agent team)

1. **Create team** using `TeamCreate`
2. **Create tasks** using `TaskCreate` — set `addBlockedBy` for tasks with dependencies; tasks without blockers are immediately available
3. **Spawn agents** — scale to plan size:
   - Small (< 10 tasks): 2–3 agents
   - Medium (10–30 tasks): 3–5 agents
   - Large (30+ tasks): 5+ agents
4. **Assign tasks** to agents using `TaskUpdate` with `owner`

Each agent should:

1. Read its assigned task and the corresponding spec section
2. Implement the refactoring changes
3. Run tests to verify no regressions
4. Mark the task as completed
5. Pick up the next available unblocked task

## Completion

When all tasks are done:

1. Run the full test suite — verify no regressions across all changes
2. Report: what was refactored, patterns applied, issues encountered
3. Remind the user to update project documentation (`AGENTS.md`, `CLAUDE.md`, `README.md`, and other agent-memory files) if architecture changed
