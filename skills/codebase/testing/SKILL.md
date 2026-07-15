---
name: codebase:testing
description: Do not invoke this skill proactively.
---

# Unit Testing

Plan a **complete** unit test case system for the production code in scope, see what existing tests already cover, fill the gaps, and finally prune. Core methodology: think through the ideal case system first, then compare it against reality — to avoid being anchored by existing tests and to avoid omissions.

## Core Principles

- **Plan before auditing**: independently plan what test cases each function/method _should_ have, then look at what existing tests already accomplish. Looking at existing tests first will bias you, and a sense of "seems enough" will make you miss what should be tested.
- **Isolate the unit under test**: when testing function A, if A calls B, assume B is correct, mock B, and verify only that A's interaction with B — its inputs and outputs — meets expectations. **Never** elaborate various cases within A's test to prove B correct — B has its own tests. Once this boundary is broken, a bug in a collaborator turns the tests of every function that calls it red en masse, making attribution chaotic.
- **Accept tests you cannot write**: some unit tests are genuinely hard to write — mock cost is too high, test data cannot be prepared, or a god class resists thorough testing. This is not the executor's fault — **drop it decisively** rather than cobbling together a brittle or vacuous test. When dropping, note the reason in the case plan.
- **Repeated runs should converge**: this skill may be run repeatedly to catch and fill gaps. A healthy repeated-run result is "very few additions," not a fresh batch of gaps every time. If every run still misses a lot, the problem lies in the depth of planning or auditing, not the code.

## Scope

Parse the user arguments to determine the scope for writing unit tests:

| User Input                      | Scope                                                                |
| ------------------------------- | -------------------------------------------------------------------- |
| No arguments                    | Whole project — plan function by function across all production code |
| Directory or package path       | That subtree — analyze all testable production code within it        |
| Specific file, class, or method | Only that target — plan cases for it                                 |

Once the scope is clear, execute Steps 1–4 in order.

## Step 1: Plan the Case System

For **each function/method** in scope, plan the test case system that belongs to it. Do **not** look at existing tests during planning.

The case plan unfolds across the following three dimensions.

### 1.1 Value Combinations of Explicit Parameters

Not an exhaustive enumeration of all permutations of parameters, but **representative** value combinations designed against the function's logic:

- **Smoke case**: a set of arguments that looks consistent with the function's expectations, without introducing challenges like zero values, empty values, or extremes, confirming the function behaves correctly on the plainest input. It is the regression baseline.
- **Zero values**: the zero value of each type (`0`, `""`, `false`, `nil`/`null`/`None`).
- **Empty values**: empty strings, empty arrays, empty collections, empty maps — various empty containers.
- **Upper/lower bounds of parameter types**: for scalar types, take their bounds, such as the max/min of `int`/`uint`/`float32`, and the positive/negative zero and precision boundaries of floats.
- **Container element count**: empty containers are already covered by "empty values"; additionally add **single-element containers** (exactly 1) and **multi-element containers** (>= 2), to cover iteration, first/last element, and off-by-one behavior.
- **Illegal input**: includes passing a non-enum member to an enum parameter, and any input that violates the function's validation contract on its parameters.

### 1.2 Value Combinations of Implicit Inputs

A function's inputs are not limited to its parameters. The following "implicit inputs" also require value-combination design:

- **Global variables**: package-level/global variables the function references, taking their different values or states.
- **Class member variables**: instance fields accessed by class methods, constructing objects in different states.
- **External calls**: other functions, methods, or network/file/database operations the function invokes. These implicit inputs are covered mainly through mocking — but note that what you cover here is not the **input** value combinations of the mocked function, but its **return value** combinations: normal return values, boundary return values, and cases that **throw errors**, to verify the function under test handles the collaborator's various outputs.

### 1.3 Logic Branches

Map the control-flow branches within the function body:

- **if/switch branches**: design value combinations (explicit parameters or mock return values) that trigger each branch.
- **Error-handling logic**: the retry, transform, propagate, degrade paths the function under test takes after catching a collaborator's error — reach them by mocking error return values.
- **Branches with no test value**: some branches have no verifiable behavior in themselves, such as merely logging a message or passing an error through unchanged. Such branches produce no output or state change that needs asserting, are not worth designing cases for, and can be skipped.
- **Hard-to-reach branches**: if a branch's trigger condition is extremely hard to construct in a unit-test environment, drop it per the "accept tests you cannot write" principle and note it.

## Step 2: Audit Existing Tests

Now look at existing tests and do two things.

### 2.1 Learn the Test Style

Figure out the codebase's unit-test conventions, so that cases you add blend in rather than stand out:

- Test naming style
- Directory and naming conventions for test files
- Test framework and organization in use (table-driven, subtests, describe blocks, etc.)
- Mock tools and mocking habits
- Assertion style, setup/teardown conventions

### 2.2 Compare Against Coverage

Take the Step 1 plan and compare it case by case against existing tests, determining which parts of the plan existing tests already cover. When judging:

- If an existing test covers the same dimension of the same function, mark it covered and do not write a new duplicate case.
- Existing tests sometimes cover scenarios the plan didn't think of — evaluate whether it is genuinely a case worth keeping; if so, fold it into the plan, otherwise record it as a deletion candidate.
- Audit must go down to the case dimension, drilling into value combinations (whether explicit or implicit), not stop at "this function has tests." A function having tests is not equivalent to it being thoroughly tested.

**Convergence self-check (critical)**: if existing tests look fairly complete yet you are still proposing a large number of new cases, stop and think again. This skill is often run repeatedly to catch and fill gaps, and the **expected result is convergence** — it should not spew a large batch of gaps on every repeated run. Honestly judge which of these situations applies:

- **Previous runs genuinely wrote too few**: then how do you ensure a one-time fill this run, so that subsequent runs converge? Fill thoroughly, covering the dimensions that should be covered, rather than tossing in a few cases perfunctorily.
- **The existing system is actually enough**: are you, to rack up work, listing a pile of cases that look useful but are actually redundant or meaningless? Delete them.
- **Insufficient understanding of the existing system**: did you fail to see that existing tests already cover certain dimensions, misjudging them as gaps? Return to 2.2 and audit more finely.

## Step 3: Fill the Missing Cases

Following the existing test style, add the cases from the plan not yet covered by existing tests.

When writing, for tests that are hard to construct (high mock cost, unpreparable data, god classes), drop them per the "accept tests you cannot write" principle — do not force a brittle test.

## Step 4: Prune Test Cases

Subtract from the tests in scope: coverage unchanged, less code:

- **Delete duplicate cases**: among multiple cases covering the same dimension of the same function, keep the most accurate one.
- **Delete valueless cases**: cases with no test meaning — for example, testing only getters/setters, assertions that always pass, testing code that no longer exists, or cases that assert incorrect behavior.
- **Compress with better idioms**: use idioms the project supports (table-driven, parameterized, shared setup/helper) to cover the same number of cases with less, more concise code. When compressing, each case must retain an identifiable label so that a failure locates the specific scenario.
- **Coverage not lost**: after pruning, the number of cases per dimension for a function under test must not fall below the dimensions covered before pruning. Pruning cuts only redundancy and verbose code, not coverage.

After pruning, run the full test suite and confirm it is green (or only intentionally skipped cases remain).
