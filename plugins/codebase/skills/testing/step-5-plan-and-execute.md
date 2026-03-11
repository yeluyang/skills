# Step 5: Plan & Execute

**Goal:** Convert the approved testing spec into a saved plan document, decompose it into parallelizable tasks with explicit dependencies, and launch agent teams to execute.

## 5.1 Save the Testing Plan

Write the approved spec from Step 4 to a plan document:

**Path:** `docs/plans/YYYY-MM-DD-testing-plan.md`

The plan document should include:

1. The full testing spec (philosophy, scope, test plan by area, statistics)
2. All resolved flagged decisions
3. The task decomposition and dependency graph (from 5.2)

Commit the plan document.

## 5.2 Task Decomposition

Break the testing spec into agent-assignable tasks. Each task should be:

- **Self-contained** — an agent can complete it without needing results from other tasks (unless blocked)
- **Scoped to one test file** — one task per test file being created or modified
- **Clear on expected output** — what test cases to write, what assertions to make

### Task Categories

All categories are **independent and parallel** — test files don't depend on each other regardless of the level of the code they test. Each task is scoped to one test file.

- **Delete** — remove existing tests marked for removal
- **Modify** — update existing tests (add assertions, rename, improve coverage)
- **Write leaf unit tests** — new tests for leaf functions (no/minimal mocking)
- **Write mid-level unit tests** — new tests for functions that call other functions (mock children)
- **Write integration tests** — new end-to-end tests across real components

## 5.3 Create Agent Team Tasks

For each task, create a task definition:

```markdown
### Task: <descriptive name>

**File:** <test file path to create or modify>

**Action:** create / modify / delete

**Test cases to implement:**
- <test case 1: name, input, expected, assertion>
- <test case 2: name, input, expected, assertion>
- ...

**Mocking requirements:** <what to mock, how>
**Test data/fixtures:** <what test data is needed>
**Verification:** Run `<test command>` — all tests in this file should pass
```

## 5.4 Launch Agent Teams

1. Create the agent team using `TeamCreate`
2. Create all tasks using `TaskCreate` — all tasks are independent, no `addBlockedBy` needed
3. Spawn agent teammates — all tasks can start immediately
4. Assign tasks to agents using `TaskUpdate` with `owner`

### Execution Strategy

- **For small plans (< 10 tasks):** 2-3 agents, tasks assigned sequentially as agents complete work
- **For medium plans (10-30 tasks):** 3-5 agents, tiered execution
- **For large plans (30+ tasks):** 5+ agents, maximum parallelism within tiers

Each agent should:

1. Read its assigned task
2. Write the test code following the project's conventions (detected in Step 0)
3. Run the tests to verify they pass (or fail as expected for TDD red-phase)
4. Commit the test file
5. Mark the task as completed
6. Pick up the next available task

## 5.5 Completion

When all tasks are completed:

1. Run the full test suite to verify no conflicts between test files
2. Report final statistics: tests added, modified, deleted, total coverage change
3. Update the progress table and proceed to Step 6
