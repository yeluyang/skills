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

After producing the overview, consult the adaptive routing table in `SKILL.md` to determine which steps apply to this project type (steps 1-7). Tell the user which steps will be executed and why, then proceed.
