# yeluyang-skills

Personal Claude Code plugin marketplace.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add yeluyang/skills
```

Then browse and install plugins:

```
/plugin install
```

## Available Plugins

| Plugin                        | Description                                                                                                                                              | Version |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| [codebase](plugins/codebase/) | Codebase understanding, documentation, and refactoring toolkit — type detection, guided walkthrough, CLAUDE.md generation, and pattern-based refactoring | 0.1.0   |

## Adding a New Plugin

1. Create a directory under `plugins/<plugin-name>/`
2. Add `.claude-plugin/plugin.json` with plugin metadata
3. Add components (`skills/`, `commands/`, `hooks/`, `agents/`, `.mcp.json`)
4. Register the plugin in `.claude-plugin/marketplace.json` at the repo root
5. Commit, push, and tag a release
