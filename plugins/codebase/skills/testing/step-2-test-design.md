# Step 2: Test Design (Bottom-Up)

**Goal:** Design tests starting from leaf functions and working upward to entry points. Focus on what the code *needs*, not what's conventional. Prioritize edge cases, boundary conditions, and anti-intuitive failure modes.

## 2.1 Design Tests for Leaf Functions

Start from every leaf function identified in Step 1. These are the simplest to test — minimal or no mocking required.

For each leaf function, design test cases across these dimensions:

### Normal Cases

- Standard, expected inputs producing expected outputs
- Common usage patterns the function was designed for

### Boundary Conditions

- **Empty / zero / null:** empty strings, empty collections, zero values, nil/null/None pointers
- **Extremes:** max int, min int, very long strings, very large collections
- **Off-by-one:** first element, last element, length boundaries, index limits
- **Type limits:** overflow, underflow, precision loss for floating point
- **Single element:** collections with exactly one item, strings with one character

### Error Cases

- Invalid input types or formats
- Values outside expected ranges
- Missing required fields
- Malformed data (corrupt, truncated, wrong encoding)
- Permission / access violations
- Timeout / cancellation scenarios

### Edge Cases

- Unicode: multi-byte characters, emoji, RTL text, zero-width characters
- Concurrency: simultaneous access, race conditions (if applicable)
- Very large input: performance edge cases, memory limits
- Unexpected but valid input: whitespace-only strings, negative-zero, NaN

### Anti-Intuitive Cases

These are the most valuable tests — scenarios that *look* correct but fail:

- Floating point comparison (`0.1 + 0.2 != 0.3`)
- String equality across encodings or normalizations
- Time zone edge cases (DST transitions, leap seconds)
- Nil vs empty collection semantics
- Default value vs explicitly-set-to-default
- Integer division truncation
- Signed/unsigned comparison traps

**For each test case, write:**

- A descriptive test name (what behavior it verifies)
- Input values
- Expected output or expected error
- Why this case matters (what bug it catches)

## 2.2 Design Tests for Mid-Level Functions

Move up the call tree to functions that call other functions.

**Key principle:** Test the *orchestration logic*, not the child behavior. Child functions already have their own tests from 2.1.

For each mid-level function:

1. **Identify what to mock** — the child functions this function calls
   - If the child is `pure` and fast → consider using real implementation (simpler)
   - If the child is `io`, `state`, or `adapter` → mock it
   - If mock setup is complex (many mock objects, complex state machines, deep dependency chains) → **flag for user decision**

2. **Design orchestration tests:**
   - Does it call children in the correct order?
   - Does it handle child errors correctly? (retry, propagate, transform?)
   - Does it pass correct arguments to children?
   - Does it combine child results correctly?

3. **Design conditional path tests:**
   - Each branch in if/switch should have a test
   - Error paths should have tests (not just happy paths)

### Mock Complexity Flag

When mock setup would require:

- More than 3 mock objects with interconnected behavior
- A mock framework the project doesn't already use
- Complex state machine simulation
- Significant test data fixtures that don't exist yet

**Flag it for the user with these options:**

1. Write the mock anyway (with estimated effort)
2. Use a lighter-touch approach (integration-style test with real dependencies)
3. Skip this specific test case
4. User provides guidance on mocking strategy

### Test Data Complexity Flag

When a test requires complex, realistic data that is hard to fabricate:

- Domain-specific data with intricate relationships (e.g., a valid order with inventory, pricing rules, discount tiers)
- Real-world samples needed to trigger edge cases (e.g., malformed PDFs, specific binary formats, production-like database snapshots)
- Large or structured fixtures that require domain knowledge to construct correctly
- Data with cross-entity consistency constraints (e.g., foreign key relationships, business invariants)

**Flag it for the user with these options:**

1. User provides sample data or points to existing fixtures the agent can adapt
2. User describes the data shape and constraints — agent generates synthetic data
3. Agent proposes a minimal data builder/factory and user validates correctness
4. Skip tests that require this data for now — add a TODO with the data requirements

## 2.3 Design Tests for Entry Points / Top-Level Functions

These are primarily integration-level tests.

For each entry point:

1. **Assess integration test feasibility:**
   - What external services, databases, or infrastructure does it need?
   - Can the system boot in a test context? What setup is required?
   - Are there existing test helpers, test databases, or Docker compose files?

2. **If feasible:** Design end-to-end tests covering:
   - The primary happy path
   - Key error paths (authentication failure, validation error, downstream service failure)
   - Data consistency (does the full flow produce correct state?)

3. **If not feasible:** Flag for user with options:
   - Set up test infrastructure (with guidance on what's needed)
   - Skip integration tests for this entry point
   - Write a partial integration test with some real + some mocked dependencies

## 2.4 Classify All Tests

For each designed test, classify:

| Type            | Description                         | Characteristics                   |
| --------------- | ----------------------------------- | --------------------------------- |
| **Unit**        | Tests one function in isolation     | Fast, mocked dependencies, no I/O |
| **Integration** | Tests a flow across real components | May need infrastructure, slower   |

## 2.5 Output

Produce the test plan organized by module/area, structured bottom-up:

```markdown
## Test Plan: <module/area name>

### Leaf-Level Unit Tests

| Function          | Test Case      | Type     | Input | Expected     | Why                            |
| ----------------- | -------------- | -------- | ----- | ------------ | ------------------------------ |
| `calculate_score` | zero input     | boundary | `0`   | `0`          | off-by-one in scoring formula  |
| `calculate_score` | negative input | error    | `-1`  | `ValueError` | no validation exists currently |
| ...               | ...            | ...      | ...   | ...          | ...                            |

### Mid-Level Unit Tests

| Function       | Test Case   | Mocking                     | Notes                      |
| -------------- | ----------- | --------------------------- | -------------------------- |
| `process_data` | happy path  | mock `fetch_record`         | verify orchestration order |
| `process_data` | fetch fails | mock `fetch_record` → error | verify error propagation   |
| ...            | ...         | ...                         | ...                        |

### Integration Tests

| Entry Point      | Test Case       | Infrastructure | Feasibility                      |
| ---------------- | --------------- | -------------- | -------------------------------- |
| `handle_request` | full happy path | DB + cache     | feasible (docker-compose exists) |
| ...              | ...             | ...            | ...                              |

### Flagged Decisions
- [ ] `process_data`: mock complexity high — user to choose approach
- [ ] `validate_order`: needs realistic order data with pricing rules — user to provide sample or confirm generated fixtures
- [ ] `handle_request` integration: needs Redis setup — user to confirm
```

After completing, update the progress table and proceed to Step 3.
