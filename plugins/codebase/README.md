# codebase

Codebase understanding, documentation, refactoring, and testing toolkit — project type detection, guided walkthrough, agent-memory generation and synchronization, pattern-based refactoring, and comprehensive test coverage analysis.

This plugin ships one shared `skills/` payload, with separate plugin metadata for Claude Code and Codex.

## Components

### Skills

- **`type`** (`$type` in Codex, `/codebase:type` in Claude Code) — Detect project type, primary languages, frameworks, and package managers by examining build files, manifests, entry points, and directory structure
- **`memory`** (`$codebase:memory` in Codex, `/codebase:memory` in Claude Code) — Deep-analyze a codebase and generate or update project memory. Defaults to `AGENTS.md`, supports agent-specific targets such as `CLAUDE.md`, and keeps sibling memory files synchronized. Uses the skill: `$type` for project detection.
- **`walkthrough`** (`$walkthrough` in Codex, `/codebase:walkthrough` in Claude Code) — Structured 7-step interactive workflow for understanding an unfamiliar codebase, with adaptive routing based on project type. Uses the skill: `$type` for project detection.
  1. Project Overview
  2. Dependencies
  3. Build & Test
  4. Core Concepts
  5. API Surface
  6. Deep Dive
  7. Quick Reference Card
- **`refactor-to-pattern`** (`$refactor-to-pattern` in Codex, `/codebase:refactor-to-pattern` in Claude Code) — Refactoring workflow: test coverage gate, responsibility analysis, SRP decomposition, cohesion vs decoupling, pattern discovery, spec-driven user review, and execution planning
- **`testing`** (`$testing` in Codex, `/codebase:testing` in Claude Code) — Analyze codebase to design and implement comprehensive test coverage — top-down code analysis, bottom-up test design, edge case focus, existing test audit, and agent team execution
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

### Codex

To make this plugin available from any working directory, clone the repo to a stable path such as `~/src/github.com/yeluyang/skills`, then register it from `~/.agents/plugins/marketplace.json` with:

```json
{
  "name": "yeluyang-skills",
  "interface": {
    "displayName": "yeluyang skills"
  },
  "plugins": [
    {
      "name": "codebase",
      "source": {
        "source": "local",
        "path": "./src/github.com/yeluyang/skills/plugins/codebase"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Coding"
    }
  ]
}
```

This lets Codex discover `codebase` from any project on that machine without copying the plugin out of the repo.

## Usage

- **Detect project type:** invoke the skill: `$type` in Codex, or `/codebase:type` in Claude Code
- **Generate project memory:** invoke the skill: `$codebase:memory` in Codex, or `/codebase:memory` in Claude Code
- **Target a specific agent:** append an agent selector, for example `$codebase:memory claude` to generate or refresh `CLAUDE.md`
- **Explore a codebase:** invoke the skill: `$walkthrough` in Codex, or `/codebase:walkthrough` in Claude Code
- **Refactor code:** invoke the skill: `$refactor-to-pattern` in Codex, or `/codebase:refactor-to-pattern` in Claude Code
- **Add test coverage:** invoke the skill: `$testing` in Codex, or `/codebase:testing` in Claude Code
