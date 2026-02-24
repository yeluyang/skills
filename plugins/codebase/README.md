# codebase

Codebase understanding, documentation, refactoring, and testing toolkit — project type detection, guided walkthrough, CLAUDE.md generation, pattern-based refactoring, and comprehensive test coverage analysis.

## Components

### Commands

- **`type`** (`/codebase:type`) — Detect project type, primary languages, frameworks, and package managers by examining build files, manifests, entry points, and directory structure

### Skills

- **`claude-md`** (`/codebase:claude-md`) — Deep-analyze a codebase and generate or update CLAUDE.md with agent-optimized project documentation. Uses `/codebase:type` for project detection.
- **`walkthrough`** (`/codebase:walkthrough`) — Structured 7-step interactive workflow for understanding an unfamiliar codebase, with adaptive routing based on project type. Uses `/codebase:type` for project detection.
  1. Project Overview
  2. Dependencies
  3. Build & Test
  4. Core Concepts
  5. API Surface
  6. Deep Dive
  7. Quick Reference Card
- **`refactor-to-pattern`** (`/codebase:refactor-to-pattern`) — Guides a 4-step refactoring process: annotate responsibilities, decompose by SRP, analyze cohesion vs decoupling, discover patterns from symptoms
- **`testing`** (`/codebase:testing`) — Analyze codebase to design and implement comprehensive test coverage — top-down code analysis, bottom-up test design, edge case focus, existing test audit, and agent team execution
  1. Code Analysis (top-down call chain tracing)
  2. Test Design (bottom-up from leaf functions)
  3. Existing Test Audit (keep/modify/delete)
  4. Spec Output & User Review
  5. Plan & Execute (agent team parallelism)

## Installation

```
/plugin marketplace add yeluyang/skills
/plugin install codebase@yeluyang-skills
```

Then restart Claude Code.

## Usage

- **Detect project type:** invoke `/codebase:type` or ask Claude to identify what kind of project this is
- **Generate CLAUDE.md:** invoke `/codebase:claude-md` or ask Claude to generate project documentation for a new codebase
- **Explore a codebase:** invoke `/codebase:walkthrough` when onboarding to an unfamiliar project
- **Refactor code:** invoke `/codebase:refactor-to-pattern` after finishing an implementation that needs architectural review
- **Add test coverage:** invoke `/codebase:testing` to analyze code and design comprehensive tests with edge case focus
