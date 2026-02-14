# Step 1: Project Overview

**Goal:** Identify what this codebase is and what it does at the highest level.

## 1.1 Detect Project Type

Invoke `/codebase:type` to detect project type, primary languages, frameworks, and package managers.

## 1.2 Identify Entry Points

Find all entry points: `main()` functions, `package.json` scripts, `Makefile` targets, CLI commands, serverless handlers, etc. List them with one-line descriptions.

## 1.3 Output

Produce a structured overview:

```markdown
## Project Overview

- **Name:** <project name>
- **Type:** <detected type>
- **Language(s):** <primary and secondary languages>
- **Entry Points:**
  - `cmd/server/main.go` — starts the HTTP/gRPC server
  - `cmd/migrate/main.go` — runs database migrations
- **Top-Level Structure:**
  - `cmd/` — entry points
  - `internal/` — application code
  - `pkg/` — shared libraries
  - ...
- **Summary:** <2-3 sentence description of what this project does>
```

## 1.4 Adaptive Routing

Based on the detected project type, determine which steps to execute:

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
