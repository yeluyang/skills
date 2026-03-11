# Step 6: Coverage Verification Loop

**Goal:** Re-analyze the codebase with fresh eyes to find coverage gaps that previous passes missed. Repeat until no significant gaps remain.

**Why this step exists:** A single analysis-and-write pass inevitably misses coverage due to context window limits and anchoring bias. Dispatching a fresh subagent for each verification round eliminates anchoring — each round sees the codebase as-is, with no memory of what the previous round focused on.

## 6.1 Verification Round

Each round follows this cycle:

```
dispatch fresh subagent → gap report → significant gaps? → write missing tests → next round
                                         ↓ no
                                       done
```

### Dispatch a Verification Subagent

Launch an **Explore** subagent with this mandate:

> Re-read all source code within the target scope. Re-read all test files within the target scope (including any just written). For every function in the source code, assess: does the test suite adequately cover it? Report any function or scenario that is uncovered or under-covered.

The subagent must:

1. **Re-read source code** — scan every file in the target scope, identify all functions/methods
2. **Re-read test files** — scan every test file, catalog what each test covers (which function, which scenario)
3. **Cross-reference** — for each function, check coverage across these dimensions:
   - Normal/happy path
   - Boundary conditions (empty, zero, extremes, off-by-one)
   - Error/failure paths
   - Edge cases and anti-intuitive scenarios
4. **Report gaps** — produce a structured gap list:

   ```markdown
   ## Verification Gap Report — Round N

   ### Uncovered Functions
   | Function | File | Why It Matters |
   | -------- | ---- | -------------- |
   | ...      | ...  | ...            |

   ### Under-Covered Functions (partial coverage exists)
   | Function | File | Missing Scenarios |
   | -------- | ---- | ----------------- |
   | ...      | ...  | ...               |

   ### Summary
   - Functions analyzed: N
   - Fully covered: X
   - Partially covered: Y
   - Uncovered: Z
   ```

### Evaluate the Gap Report

**Significant gaps** — proceed to write missing tests:

- Any function with zero test coverage
- Any function missing error path or boundary coverage
- Any scenario gap that could mask a real bug

**Trivial or empty** — exit the loop:

- All functions have tests covering normal, boundary, and error dimensions
- Remaining gaps are minor (e.g., cosmetic naming, a rare platform-specific edge case)
- The subagent explicitly states no significant gaps remain

### Write Missing Tests

If significant gaps exist:

1. For small gap sets (< 5 test files affected): write the tests directly in the current session
2. For larger gap sets: decompose into tasks and launch agent teams (same approach as Step 5)
3. Run the test suite to verify all new tests pass
4. Increment the round counter

## 6.2 Loop Control

- **Maximum rounds:** 3 (configurable — if the user requests more, honor it)
- **Round counter:** track and display at each iteration

After each round, report:

```markdown
### Verification Round N Complete

- Gaps found: X
- Tests added: Y
- Tests modified: Z
- Remaining gaps: W (if any — list them)
```

### Early Exit

Exit before the max round limit if:

- A round's gap report finds zero significant gaps
- A round finds only gaps that were already flagged and deferred by the user in Step 4

### Max Rounds Reached

If the loop hits the cap and gaps remain:

1. List all remaining gaps with their severity
2. Ask the user: continue for more rounds, or accept current coverage and move on?

## 6.3 Completion

When the loop exits (either clean or user-approved):

1. Report final verification statistics:
   - Total rounds executed
   - Total tests added across all rounds
   - Coverage status: which areas are now fully covered
2. Update the progress table and proceed to Step 7
