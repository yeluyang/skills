# codebase

Codebase understanding, documentation, refactoring, and testing toolkit — project type detection, guided walkthrough, agent-memory generation and synchronization, pattern-based refactoring, and comprehensive test coverage analysis.

This plugin ships one shared `skills/` payload with a Claude Code manifest, while keeping optional per-agent helpers under `skills/*/agents/*.yaml` for `vercel-labs/skills` users.

## Components

### Skills

- **`codebase:type`** (`/codebase:type` in Claude Code; skill name `codebase:type` in `vercel-labs/skills`) — Detect project type, primary languages, frameworks, and package managers by examining build files, manifests, entry points, and directory structure.
- **`codebase:memory`** (`/codebase:memory` in Claude Code; skill name `codebase:memory`) — Deep-analyze a codebase and generate or update project memory. Defaults to `AGENTS.md`, supports agent-specific targets such as `CLAUDE.md`, and keeps sibling memory files synchronized. Uses the skill `codebase:type` for project detection.
- **`codebase:walkthrough`** (`/codebase:walkthrough` in Claude Code; skill name `codebase:walkthrough`) — Structured 7-step interactive workflow for understanding an unfamiliar codebase, with adaptive routing based on project type. Uses the skill `codebase:type` for project detection.
  1. Project Overview
  2. Dependencies
  3. Build & Test
  4. Core Concepts
  5. API Surface
  6. Deep Dive
  7. Quick Reference Card
- **`codebase:refactor-to-pattern`** (`/codebase:refactor-to-pattern` in Claude Code; skill name `codebase:refactor-to-pattern`) — Refactoring workflow: test coverage gate, responsibility analysis, SRP decomposition, cohesion vs decoupling, pattern discovery, spec-driven user review, and execution planning.
- **`codebase:testing`** (`/codebase:testing` in Claude Code; skill name `codebase:testing`) — Analyze codebase to design and implement comprehensive test coverage — top-down code analysis, bottom-up test design, edge case focus, existing test audit, and agent team execution.
  1. Code Analysis (top-down call chain tracing)
  2. Test Design (bottom-up from leaf functions)
  3. Existing Test Audit (keep/modify/delete)
  4. Spec Output & User Review
  5. Plan & Execute (agent team parallelism)
  6. Coverage Verification (fresh-subagent gap detection loop)
  7. Test Consolidation (reduce redundancy without losing coverage)

## Installation

### Claude Code

```text
/plugin marketplace add yeluyang/skills
/plugin install codebase@yeluyang-skills
```

Then restart Claude Code.

### vercel-labs/skills

```bash
npx skills add yeluyang/skills --list
npx skills add yeluyang/skills --skill codebase:type --skill codebase:memory -a claude-code
```

To target other agents, replace `-a claude-code` with your agent id.

## Usage

- **Detect project type (Claude Code):** `/codebase:type`
- **Generate project memory (Claude Code):** `/codebase:memory`
- **Target a specific memory file:** append an agent selector, for example `codebase:memory claude` to generate or refresh `CLAUDE.md`.
- **Explore a codebase (Claude Code):** `/codebase:walkthrough`
- **Refactor code (Claude Code):** `/codebase:refactor-to-pattern`
- **Add test coverage (Claude Code):** `/codebase:testing`
- **Other agents via `vercel-labs/skills`:** invoke skill names (`codebase:type`, `codebase:memory`, `codebase:walkthrough`, `codebase:refactor-to-pattern`, `codebase:testing`) in your agent workflow.
