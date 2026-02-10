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

| Plugin                                                | Description                                                                                                                                       | Version |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| [hello-world](plugins/hello-world/)                   | Sample skill to verify installation works                                                                                                         | 0.1.0   |
| [claude-md](plugins/claude-md/)                       | Generate or update CLAUDE.md by deep-analyzing a codebase to extract stable knowledge and operational details into agent-optimized documentation  | 0.1.0   |
| [codebase-walkthrough](plugins/codebase-walkthrough/) | Guided codebase understanding — detect project type, map dependencies, extract core concepts, enumerate API surfaces, and deep-dive interactively | 0.1.0   |
| [refactor-to-pattern](plugins/refactor-to-pattern/)   | Refactoring workflow — evaluate code with DRY + SOLID, decompose by SRP, organize by architectural layers, discover design patterns organically   | 0.1.0   |

## Adding a New Plugin

1. Create a directory under `plugins/<plugin-name>/`
2. Add `.claude-plugin/plugin.json` with plugin metadata
3. Add components (`skills/`, `commands/`, `hooks/`, `agents/`, `.mcp.json`)
4. Register the plugin in `.claude-plugin/marketplace.json` at the repo root
5. Commit, push, and tag a release
