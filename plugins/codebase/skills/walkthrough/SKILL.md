---
name: codebase:walkthrough
description: Guided codebase understanding — detect project type, map dependencies and external systems, extract core concepts, enumerate API surfaces, and deep-dive into request paths interactively
---

# Codebase Walkthrough

Structured, interactive workflow for understanding an unfamiliar codebase. Adapts analysis depth to the project type — from simple CLIs to complex network services.

## When to Use

Invoke this skill when:

- Onboarding to an unfamiliar codebase
- Exploring a new project before contributing
- Building a mental model of how a system works
- Preparing to do a major refactor or feature addition in code you don't yet understand

## Progress Tracker

After completing each step, output a progress table so the user knows where they are:

```markdown
| Step                    | Status    |
| ----------------------- | --------- |
| 1. Project Overview     | done      |
| 2. Dependencies         | done      |
| 3. Build & Test         | ▶ current |
| 4. Core Concepts        | —         |
| 5. API Surface          | —         |
| 6. Deep Dive            | —         |
| 7. Quick Reference Card | —         |
```

Update this table at the end of every step.

## Workflow

Execute steps in order. Each step is documented in its own file — **read the step file before executing that step**.

| Step | File                         | Goal                                                     |
| ---- | ---------------------------- | -------------------------------------------------------- |
| 1    | `step-1-project-overview.md` | Detect project type, entry points, top-level structure   |
| 2    | `step-2-dependencies.md`     | Map frameworks, libraries, external systems              |
| 3    | `step-3-build-test.md`       | Build, run, test commands                                |
| 4    | `step-4-core-concepts.md`    | Domain entities, schema, abstractions, design philosophy |
| 5    | `step-5-api-surface.md`      | Enumerate APIs, highlight core paths                     |
| 6    | `step-6-deep-dive.md`        | Path selection, end-to-end tracing, Q&A loop             |
| 7    | `step-7-quick-reference.md`  | Generate condensed reference card                        |

### Adaptive Routing

Based on detected project type in Step 1, adjust which steps are executed:

| Type                 | Steps to Execute                                           |
| -------------------- | ---------------------------------------------------------- |
| API Protocol / IDL   | 1 → 4 (schemas as concepts) → 5 (endpoint listing) → 7     |
| Documentation Site   | 1 → 3 (build) → 7                                          |
| CLI (single-run)     | 1 → 2 → 3 → 4 → 6 (trace main flow directly) → 7           |
| CLI (daemon)         | 1 → 2 → 3 → 4 → 5 (commands/events) → 6 → 7                |
| Web Frontend         | 1 → 2 → 3 → 4 → 5 (routes/pages) → 6 → 7                   |
| Mobile App           | 1 → 2 → 3 → 4 → 5 (screens/navigation) → 6 → 7             |
| Desktop App          | 1 → 2 → 3 → 4 → 5 (windows/IPC) → 6 → 7                    |
| Network Service      | 1 → 2 → 3 → 4 → 5 → 6 → 7 (full workflow)                  |
| SDK / Library        | 1 → 2 → 3 → 4 → 5 (public API surface) → 6 → 7             |
| Data Pipeline / ETL  | 1 → 2 → 3 → 4 → 5 (pipeline stages) → 6 → 7                |
| ML / AI Project      | 1 → 2 → 3 → 4 → 5 (training/inference flows) → 6 → 7       |
| Infrastructure / IaC | 1 → 2 → 3 → 4 (resources & modules) → 7                    |
| Plugin / Extension   | 1 → 2 → 3 → 4 → 5 (activation events/commands) → 6 → 7     |
| Embedded / Firmware  | 1 → 2 → 3 → 4 → 5 (interrupt handlers/peripherals) → 6 → 7 |
| Game                 | 1 → 2 → 3 → 4 → 5 (game loop/systems) → 6 → 7              |

Tell the user which steps will be executed and why, then proceed.

## Guiding Principles

- **Show, don't tell** — always reference specific files and line numbers when describing code
- **Structured output** — use markdown tables, code blocks, and ASCII diagrams; avoid walls of prose
- **Breadth before depth** — map the whole surface (Steps 1-5) before diving into any single path (Step 6)
- **User-driven depth** — never deep-dive without asking; let the user choose what matters to them
- **Confirm assumptions** — after detecting project type and core concepts, verify with the user before proceeding
- **Complexity heatmap** — when relevant, mention which directories/files are the largest or most complex, so the user knows where the "heavy" code lives
