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
- **Skill dependency rules** — when a skill (`SKILL.md`) decomposes into sub-files (`step-*.md`, `part-*.md`, etc.):
  - Dependencies flow **parent → child only**; sub-files must never reference the parent `SKILL.md`
  - Sibling sub-files must not reference each other directly
  - Data flows through the parent: parent executes child A, receives output, passes it to child B
  - If a sub-file needs material from a sibling, that coupling is wrong — extract the dependent part into its own sub-file and let the parent orchestrate the ordering
- **Commands must not block when invoked by skills** — commands (e.g., `/codebase:type`) may be called by skills as sub-steps; avoid prompts like "confirm with user before proceeding" that stall automated workflows

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

## Updating Skills

When modifying any skill (`SKILL.md`), command, or other plugin content:

1. **Bump the plugin version** — update `version` in both:
   - `plugins/<name>/.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` (the corresponding entry)
2. **Keep versions in sync** — both files must show the same version for the plugin.
3. **Use semver**:
   - Patch (`0.1.2` → `0.1.3`): wording tweaks, clarifications, minor prompt adjustments
   - Minor (`0.1.3` → `0.2.0`): new skill/command added, significant workflow changes, new sections
   - Major (`0.2.0` → `1.0.0`): breaking changes to skill behavior or plugin structure

This is easy to forget — **treat the version bump as part of the edit, not a separate step.**

## Commands

No build, test, or lint steps — this is a content-only repository. Validate by inspecting JSON/Markdown structure manually or by installing into Claude Code and testing.
