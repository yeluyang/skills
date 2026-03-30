# yeluyang-skills

Personal plugin marketplace for Claude Code and Codex.

This repo stays content-only: shared `skills/` and `commands/` live once under `plugins/<name>/`, while each product gets its own metadata surface.

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

### Codex

Use this repo from Codex through your machine-local personal marketplace. Clone the repo to a stable path on each machine, for example:

```text
~/src/github.com/yeluyang/skills
```

Then create `~/.agents/plugins/marketplace.json` and point it at the cloned repo:

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

Because the personal marketplace file lives at `~/.agents/plugins/marketplace.json`, the plugin path is resolved relative to your home directory. On a new machine, clone the repo to the same location and `git pull` to update it.

## Available Plugins

| Plugin                        | Description                                                                                                                                                                               | Version |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| [codebase](plugins/codebase/) | Codebase understanding, documentation, refactoring, and testing toolkit — project type detection, guided walkthrough, CLAUDE.md generation, pattern-based refactoring, and test analysis | 0.6.0   |

## Repo Layout

```text
.claude-plugin/marketplace.json   Claude Code marketplace index
plugins/<name>/
  .claude-plugin/plugin.json      Claude Code plugin manifest
  .codex-plugin/plugin.json       Codex plugin manifest
  skills/                         Shared skill payload
  commands/                       Shared slash commands
  README.md                       Human-facing plugin docs
```

Codex personal marketplace usage references the plugin directories directly from `~/.agents/plugins/marketplace.json`.

## Adding a New Plugin

1. Create `plugins/<plugin-name>/`.
2. Add both plugin manifests:
   `plugins/<plugin-name>/.claude-plugin/plugin.json`
   `plugins/<plugin-name>/.codex-plugin/plugin.json`
3. Add shared components as needed: `skills/`, `commands/`, `hooks/`, `agents/`, `.mcp.json`.
4. Register the plugin:
   - in `.claude-plugin/marketplace.json` for Claude Code
   - in `~/.agents/plugins/marketplace.json` on each machine for Codex personal marketplace
5. Keep shared metadata aligned across both manifests: `name`, `version`, `description`, `author`, `repository`, `license`.
6. Commit, push, and tag a release.
