# CLAUDE.md

## Overview

Personal Claude Code plugin marketplace. Hosts plugins (skills, commands, hooks, agents) that can be installed into Claude Code via `/plugin marketplace add yeluyang/skills`. The repo is content-only — no build steps, no runtime code, just structured Markdown and JSON metadata.

## Architecture

```
.claude-plugin/marketplace.json   ← Registry: lists all plugins with name, version, source path
plugins/<name>/                   ← One directory per plugin
  .claude-plugin/plugin.json      ← Plugin metadata (name, version, author, license)
  skills/<skill>/SKILL.md         ← Skill definitions (frontmatter + prompt)
  skills/<skill>/<aux>.md         ← Optional supplementary files (step guides, references)
  commands/<command>.md            ← Slash commands (frontmatter + prompt)
  README.md                       ← Human-facing docs for the plugin
```

Plugins are self-contained — each `plugins/<name>/` directory holds everything Claude Code needs to load that plugin. The root `marketplace.json` is the index that ties them together.

Complex skills can include supplementary Markdown files alongside `SKILL.md` in the same directory (e.g., `walkthrough` has step-1 through step-7 guides that the main skill references).

## Conventions

- Plugin directories use kebab-case: `plugins/codebase/`
- Skill directories match the skill name: skill `claude-md` lives in `skills/claude-md/SKILL.md`
- Each plugin has its own `plugin.json` with `name`, `version`, `description`, `author`, `repository`, `license`
- Skills use YAML frontmatter (`name`, `description`) followed by Markdown prompt body
- Commands use YAML frontmatter (`description`) followed by Markdown prompt body
- Skills are triggered by user intent detection; commands are explicit slash commands (`/codebase:type`)
- Versioning is semver (`0.1.0`)

## How to Add

**To add a new plugin:**

1. Create `plugins/<plugin-name>/` directory
2. Create `.claude-plugin/plugin.json` with metadata (`name`, `version`, `description`, `author`, `repository`, `license`)
3. Add components as needed:
   - `skills/<skill-name>/SKILL.md` — skill with YAML frontmatter and Markdown body
   - `skills/<skill-name>/<aux>.md` — optional supplementary files for complex skills
   - `commands/<command-name>.md` — slash command with YAML frontmatter and Markdown body
   - `hooks/`, `agents/`, `.mcp.json` — as required
4. Create `README.md` documenting the plugin's components and usage
5. Register in root `.claude-plugin/marketplace.json` — add entry to `plugins` array with `name`, `description`, `version`, `source` (relative path to plugin dir), `author`
6. Commit, push, tag a release

## Commands

No build, test, or lint steps — this is a content-only repository. Validate by inspecting JSON/Markdown structure manually or by installing into Claude Code and testing.
