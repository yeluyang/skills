# codebase

Codebase understanding, refactoring, and testing toolkit — project type detection, guided walkthrough, pattern-based refactoring, and comprehensive test coverage analysis.

This plugin ships one shared `skills/` payload with a Claude Code manifest, while keeping optional per-agent helpers under `skills/*/agents/*.yaml` for `vercel-labs/skills` users.

## Components

### Skills

- **`codebase:type`** (`/codebase:type` in Claude Code; skill name `codebase:type` in `vercel-labs/skills`) — Detect project type, primary languages, frameworks, and package managers by examining build files, manifests, entry points, and directory structure.
- **`codebase:walkthrough`** (`/codebase:walkthrough` in Claude Code; skill name `codebase:walkthrough`) — Structured 7-step interactive workflow for understanding an unfamiliar codebase, with adaptive routing based on project type. Uses the skill `codebase:type` for project detection.
  1. Project Overview
  2. Dependencies
  3. Build & Test
  4. Core Concepts
  5. API Surface
  6. Deep Dive
  7. Quick Reference Card
- **`codebase:refactor-to-pattern`** (`/codebase:refactor-to-pattern` in Claude Code; skill name `codebase:refactor-to-pattern`) — Refactoring workflow: test coverage gate, responsibility analysis, SRP decomposition, cohesion vs decoupling, pattern discovery, spec-driven user review, and execution planning.
- **`codebase:testing`** (`/codebase:testing` in Claude Code; skill name `codebase:testing`) — Plan a complete unit test case system for production code, audit what existing tests already cover, fill the gaps, then prune redundancy.
  1. Plan the test case system (params, implicit inputs, branches)
  2. Audit existing tests (style + coverage)
  3. Fill the missing cases
  4. Prune redundant and valueless cases

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
npx skills add yeluyang/skills --skill codebase:type --skill codebase:walkthrough -a claude-code
```

To target other agents, replace `-a claude-code` with your agent id.

## Usage

- **Detect project type (Claude Code):** `/codebase:type`
- **Explore a codebase (Claude Code):** `/codebase:walkthrough`
- **Refactor code (Claude Code):** `/codebase:refactor-to-pattern`
- **Add test coverage (Claude Code):** `/codebase:testing`
- **Other agents via `vercel-labs/skills`:** invoke skill names (`codebase:type`, `codebase:walkthrough`, `codebase:refactor-to-pattern`, `codebase:testing`) in your agent workflow.
