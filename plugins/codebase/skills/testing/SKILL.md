---
name: codebase:testing
description: Analyze codebase to design and implement comprehensive test coverage — top-down code analysis, bottom-up test design, edge case focus, existing test audit, and agent team execution
---

# Testing

Analyze a codebase to understand what tests it _needs_, audit what it already _has_, and execute a plan to close the gap. Philosophy: understand the code's testing requirements independently before looking at existing tests — prevents anchoring bias.

## When to Use

Invoke this skill when:

- Adding test coverage to an under-tested codebase
- Reviewing and improving existing test quality
- Starting work on a new module and wanting thorough test design upfront
- Auditing test coverage to find gaps, especially edge cases and boundary conditions

## Scope Resolution

The skill adapts to what the user provides:

| User Input                      | Scope                                                            |
| ------------------------------- | ---------------------------------------------------------------- |
| No arguments                    | Whole project — identify all logical entry points, full analysis |
| Directory or package path       | That subtree — analyze all testable code within                  |
| Specific file, class, or method | That target — trace its call chain, analyze from there           |

## Progress Tracker

After completing each step, output a progress table:

```markdown
| Step                     | Status    |
| ------------------------ | --------- |
| 1. Code Analysis         | done      |
| 2. Test Design           | ▶ current |
| 3. Existing Test Audit   | —         |
| 4. Spec & Review         | —         |
| 5. Plan & Execute        | —         |
| 6. Coverage Verification | —         |
| 7. Test Consolidation    | —         |
```

Update this table at the end of every step.

## Workflow

Execute steps in order. Each step is documented in its own file — **read the step file before executing that step**.

| Step | File                           | Goal                                                                |
| ---- | ------------------------------ | ------------------------------------------------------------------- |
| 0    | _(this file)_                  | Detect scope, invoke `/codebase:type`, route to steps               |
| 1    | `step-1-code-analysis.md`      | Top-down: trace call chains, identify leaf functions, categorize    |
| 2    | `step-2-test-design.md`        | Bottom-up: design tests from leaves upward, edge cases, flags       |
| 3    | `step-3-existing-audit.md`     | Audit existing tests: keep, modify, delete, identify gaps           |
| 4    | `step-4-spec-output.md`        | Synthesize testing spec, present to user, iterate until approved    |
| 5    | `step-5-plan-and-execute.md`   | Save plan, decompose into tasks, launch agent teams                 |
| 6    | `step-6-verification-loop.md`  | Re-analyze with fresh subagents, find missed gaps, loop until clean |
| 7    | `step-7-test-consolidation.md` | Reduce redundancy and structural bloat without losing coverage      |

### Step 0: Scope Detection & Project Type

Before entering the step files:

1. **Parse user arguments** to determine scope (see Scope Resolution table above)
2. **Invoke `/codebase:type`** to detect project type, languages, frameworks, and test tooling
3. **Identify test conventions** from project detection: test runner, test file naming, test directory structure, assertion libraries, mock frameworks
4. **Adapt step depth** based on scope size:
   - Single function/class → steps are lightweight; may produce compact output
   - Whole project → full multi-step workflow with user checkpoints

Log scope and detection results, then proceed to Step 1.

## Guiding Principles

- **Tests-needed-first** — understand what the code requires before looking at what exists; prevents anchoring bias
- **Bottom-up construction** — leaf functions are simplest to test; patterns established there guide higher-level tests
- **Edge cases over happy paths** — boundary conditions, anti-intuitive cases, and hidden failure modes are where real bugs live
- **User at decision points** — don't guess on complex mocking strategies or integration test feasibility; ask
- **Independence maximized** — unit tests should be as isolated and fine-grained as possible
- **Parallel execution** — decompose test-writing tasks for agent team parallelism where possible
