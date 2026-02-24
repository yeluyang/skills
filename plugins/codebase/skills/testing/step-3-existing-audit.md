# Step 3: Existing Test Audit

**Goal:** NOW look at existing tests. Map them against the test plan from Step 2 to identify what can be reused, what needs modification, and what should be deleted. Minimize work by leveraging quality existing tests.

**Principle:** You designed the test plan in Steps 1-2 without looking at existing tests — that was deliberate, to prevent anchoring bias. Now compare your ideal plan against reality.

## 3.1 Scan Existing Tests

Find all test files within the target scope:

- Use the project's test file naming conventions detected in Step 0 (e.g., `*_test.go`, `*.test.ts`, `test_*.py`)
- Include any test helper files, fixtures, or test utilities
- Note the test framework and assertion library in use

For each test file, read it and catalog:

- Which functions/classes it tests
- What test cases it contains
- What mocking approach it uses
- Test quality indicators: assertion specificity, descriptive names, proper setup/teardown

## 3.2 Map Existing Tests to Plan

For each test case in the Step 2 test plan, check if an existing test covers it:

### Keep (exact or close match)

The existing test covers the same function and scenario with adequate assertions.

- Mark the planned test as **covered**
- Remove it from the to-write list
- Note the existing test's location for reference

### Modify (partial match)

The existing test covers the right function but:

- Misses edge cases that Step 2 identified
- Has weak assertions (e.g., only checks for no-error, doesn't verify output)
- Uses outdated patterns or deprecated APIs
- Has poor naming that obscures intent

Document the specific modifications needed.

### Delete (no value or harmful)

The existing test should be removed because:

- It tests code that no longer exists (dead test)
- It tests trivial behavior that provides no real coverage (e.g., testing getters/setters)
- It is fundamentally broken (always passes, wrong assertions, tests implementation details that change constantly)
- It duplicates another test without adding value
- It actively misleads by asserting incorrect behavior

### Add (existing test reveals gap)

Sometimes an existing test covers a scenario you didn't think of in Step 2:

- Evaluate: is this a real edge case worth testing?
- If yes → add it to the test plan
- If no → mark the existing test as a delete candidate

## 3.3 Quality Assessment

For all tests marked **keep** or **modify**, evaluate:

| Quality Dimension  | Good                                                      | Needs Improvement                     |
| ------------------ | --------------------------------------------------------- | ------------------------------------- |
| **Assertions**     | Specific value checks, error type checks                  | Only checks `!= nil` or `!= null`     |
| **Naming**         | Describes behavior: `test_returns_error_when_input_empty` | Vague: `test_function1`, `test_basic` |
| **Isolation**      | Each test is independent, no shared mutable state         | Tests depend on execution order       |
| **Setup/Teardown** | Clean, uses helpers or fixtures                           | Manual setup repeated in every test   |
| **Readability**    | Arrange-Act-Assert structure, clear intent                | Logic buried in helpers, unclear flow |

Note quality issues as part of the **modify** recommendations.

## 3.4 Output

Produce the audit report:

```markdown
## Existing Test Audit

### Summary
- Total existing tests scanned: N
- Keep as-is: X
- Modify: Y (with specific changes)
- Delete: Z (with reasons)
- New tests revealed by audit: W

### Keep (no changes needed)
| Existing Test                | Covers Planned Test           | Location                   |
| ---------------------------- | ----------------------------- | -------------------------- |
| `test_calculate_score_basic` | `calculate_score` normal case | `tests/test_scoring.py:15` |
| ...                          | ...                           | ...                        |

### Modify (specific changes needed)
| Existing Test       | Changes Needed                                                                | Location                   |
| ------------------- | ----------------------------------------------------------------------------- | -------------------------- |
| `test_process_data` | Add assertion for error propagation; rename to `test_process_data_happy_path` | `tests/test_process.py:42` |
| ...                 | ...                                                                           | ...                        |

### Delete (with rationale)
| Existing Test      | Reason                 | Location                    |
| ------------------ | ---------------------- | --------------------------- |
| `test_old_handler` | Tests removed endpoint | `tests/test_handlers.py:88` |
| ...                | ...                    | ...                         |

### Remaining To-Write (gap analysis)
| Planned Test                               | Priority | Notes                                    |
| ------------------------------------------ | -------- | ---------------------------------------- |
| `calculate_score` boundary: negative input | high     | no existing coverage for negative values |
| `process_data` error path: fetch timeout   | medium   | error paths untested                     |
| ...                                        | ...      | ...                                      |
```

After completing, update the progress table and proceed to Step 4.
