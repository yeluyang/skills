# Step 1: Code Analysis (Top-Down)

**Goal:** Understand the target code's structure and call relationships before thinking about tests. Build a dependency map from entry points down to leaf functions.

## 1.1 Identify Entry Points

Based on the scope determined in Step 0:

**Whole project:**

- Find main functions, API handlers, CLI commands, exported public APIs
- Find scheduled tasks, event handlers, message consumers
- Find any code registered as callbacks or hooks

**Directory / package:**

- Find all public functions, classes, and methods in that package
- Include any package-level init functions

**Specific target:**

- Start from the named function, class, or method
- This is your sole entry point

List each entry point with its file path and a one-line description of what it does.

## 1.2 Recursive Call Chain Tracing

For each entry point, trace its call chain recursively:

1. Read the function body
2. Identify every function/method it calls (excluding stdlib/third-party)
3. For each called function, repeat from (1)
4. Stop when reaching a **leaf function**

**Leaf function** — a function where the tracing stops:

- Contains only its own logic with no further project-internal calls, OR
- Only calls stdlib, third-party libraries, or external SDKs — everything else is its own logic

Build the call tree. Note the depth of each function from its entry point.

## 1.3 Function Categorization

Tag every function in the call tree with its category:

| Category       | Description                                           | Testing Implication                                |
| -------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `pure`         | No side effects, deterministic output for given input | Easiest to test — deterministic, no dependencies   |
| `io`           | Performs file, network, or database operations        | Needs mocking or test fixtures                     |
| `state`        | Mutates shared/global state                           | Needs careful setup/teardown isolation             |
| `orchestrator` | Mainly coordinates other functions, minimal own logic | Test the coordination logic, isolate from children |
| `adapter`      | Wraps external APIs or SDKs, translates data formats  | Isolate from external system, test the translation |

A function may have multiple categories (e.g., `io` + `adapter`). Note all that apply.

## 1.4 Output

Produce a structured call graph for each entry point:

```markdown
## Call Graph: <entry point name>

**Entry:** `path/to/file:function_name` — <description>

```

entry_function [orchestrator]
├── validate_input [pure]
├── fetch_record [io, adapter]
│   └── parse_response [pure]  ← leaf
├── process_data [pure]
│   ├── calculate_score [pure]  ← leaf
│   └── apply_rules [pure]
│       └── check_threshold [pure]  ← leaf
└── save_result [io]  ← leaf

```

**Leaf functions:** <list with file paths>
**External dependencies:** <list of stdlib/third-party calls per function>
**Complexity hotspots:** <functions with high branching, deep nesting, or many dependencies>
```

After completing, update the progress table and proceed to Step 2.
