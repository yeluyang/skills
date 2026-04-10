# yeluyang-skills

Personal plugin marketplace for Claude Code, with shared skill payloads that are also consumable by `vercel-labs/skills`.

This repo stays content-only: shared `skills/` and `commands/` live once under `plugins/<name>/`.

## Installation

### Claude Code

Add this marketplace:

```text
/plugin marketplace add yeluyang/skills
```

Then install a plugin:

```text
/plugin install codebase@yeluyang-skills
```

### vercel-labs/skills

Install directly from this repository with the open `skills` CLI:

```bash
npx skills add yeluyang/skills --list
npx skills add yeluyang/skills --skill codebase:type -a claude-code
```

Use `-a <agent>` to target any supported agent (`claude-code`, `codex`, `cursor`, etc.).

## Available Plugins

| Plugin                        | Description                                                                                                                                                                                                     | Version |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| [codebase](plugins/codebase/) | Codebase understanding, documentation, refactoring, and testing toolkit — project type detection, guided walkthrough, agent-memory generation and synchronization, pattern-based refactoring, and test analysis | 0.9.0   |
| [git](plugins/git/)           | Git workflows and repository operations toolkit                                                                                                                                                                 | 0.3.0   |

## Repo Layout

```text
.claude-plugin/marketplace.json           Claude Code marketplace index
plugins/<name>/
  .claude-plugin/plugin.json              Claude Code plugin manifest
  skills/                                 Shared skill payload
  skills/<skill>/agents/<provider>.yaml   Optional per-agent helper prompt
  commands/                               Shared slash commands
  README.md                               Human-facing plugin docs
```

`vercel-labs/skills` discovers installable skills from repository structure and `SKILL.md` metadata; no `.codex-plugin/plugin.json` files are required.

## Adding a New Plugin

1. Create `plugins/<plugin-name>/`.
2. Add the Claude Code plugin manifest:
   `plugins/<plugin-name>/.claude-plugin/plugin.json`
3. Add shared components as needed: `skills/`, `commands/`, `hooks/`, `agents/`, `.mcp.json`.
4. Register the plugin in `.claude-plugin/marketplace.json`.
5. Keep shared metadata aligned between `plugins/<plugin-name>/.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (`name`, `version`, `description`, `author`, `repository`, `license`).
6. Update `README.md` and `plugins/<plugin-name>/README.md` if install or usage surfaces changed.
7. Commit, push, and tag a release.
