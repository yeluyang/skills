# Step 7: Test Consolidation

**Goal:** Review all tests in the target scope for redundancy and structural bloat. Consolidate aggressively — but never at the cost of coverage.

**Why this step exists:** Tests accumulate over time from many sources — different authors, agents, sessions, or rounds of work — and redundancy creeps in naturally. It can exist between newly written tests, between pre-existing tests, or across the two groups. All three categories must be checked. Consolidation reduces maintenance burden without losing the coverage gains.

## 7.1 Identify Consolidation Opportunities

Read every test file in the target scope. For each file, and across files, look for these patterns. This list is illustrative, not exhaustive — use judgment to find any consolidation opportunity that reduces redundancy without losing coverage.

### Redundant Tests

Tests that cover the same function, same scenario, with equivalent assertions. One adds no coverage the other doesn't already provide.

**Action:** keep the more descriptive / more thorough one, delete the other.

### Repetitive-Input Tests

Multiple tests with identical structure — same setup flow, same assertion pattern — differing only in input values and expected output.

**Action:** consolidate into a single test that iterates over all cases. Choose the technique best suited to the project's language, test framework, and conventions — the goal is reducing structural repetition while keeping every case individually identifiable in failure output.

### Subset Tests

Test A asserts a strict subset of what test B asserts (same function, same scenario, fewer checks). Test A provides no additional coverage beyond test B.

**Action:** delete test A.

### Repeated Setup

Multiple tests repeat the same setup code verbatim (constructing objects, initializing state, configuring mocks).

**Action:** extract into a shared mechanism — but only when the repeated setup appears 3+ times AND extracting it improves readability. Do not extract for the sake of DRY alone.

### Parallel-Structure Tests

Tests that follow the same flow pattern with minor variations (e.g., "create resource → modify → verify" repeated with different resource types or configurations).

**Action:** consolidate using whatever technique the language and framework support for parameterizing test flows. Each consolidated case must retain a descriptive label.

### Anything Else

The patterns above are common, not comprehensive. The general principle: if two or more tests can be combined or restructured to reduce code volume without losing any coverage information, do it.

## 7.2 Coverage Preservation Constraint

**This is non-negotiable.** Consolidation must never cause:

- **Code path loss** — every function, branch, and code path that was covered before consolidation must still be covered after
- **Scenario loss** — every normal, boundary, error, edge, and anti-intuitive case must still be tested
- **Assertion weakening** — do not replace specific value assertions with weaker checks (e.g., replacing `assertEqual(result, 42)` with `assertNotNone(result)`)
- **Label loss** — when merging multiple cases into one test, every case must have a descriptive name/label so that failures are traceable to the specific scenario

### Pre/Post Verification

1. **Before consolidation:** run the full test suite, record:
   - Total test count
   - All passing/failing
   - If the project has a coverage tool: record coverage percentage and covered lines
2. **After consolidation:** run the full test suite again, verify:
   - All tests still pass
   - No previously-covered code paths became uncovered (if coverage tool available)
   - Review consolidated tests to confirm every original scenario is still represented

If any post-consolidation test fails or coverage drops, revert that specific consolidation and investigate.

## 7.3 Execution

1. **Catalog** — list all consolidation opportunities with their category
2. **Estimate impact** — for each opportunity, note: N tests consolidated into M, estimated lines saved
3. **Present summary to user:**

   ```markdown
   ## Consolidation Plan

   | Category           | Instances | Tests Before | Tests After | Lines Saved (est.) |
   | ------------------ | --------- | ------------ | ----------- | ------------------ |
   | Redundant          | N         | X            | Y           | ~Z                 |
   | Repetitive-input   | N         | X            | Y           | ~Z                 |
   | Subset             | N         | X            | Y           | ~Z                 |
   | Repeated setup     | N         | —            | —           | ~Z                 |
   | Parallel-structure | N         | X            | Y           | ~Z                 |
   | Other              | N         | X            | Y           | ~Z                 |
   | **Total**          | **N**     | **X**        | **Y**       | **~Z**             |
   ```

4. **Get user approval** — ask: proceed with all consolidations, review individually, or skip?
5. **Execute** — apply consolidations, run test suite after each file is modified to catch regressions early
6. **Report results:**

   ```markdown
   ## Consolidation Results

   - Tests before: N
   - Tests after: M
   - Net reduction: N - M
   - Lines before: X
   - Lines after: Y
   - Net reduction: X - Y
   - All tests passing: yes/no
   - Coverage change: none / +N% / -N% (if measurable)
   ```

## 7.4 Completion

1. Verify all tests pass one final time
2. Report the final test suite statistics
3. Update the progress table — all steps done
