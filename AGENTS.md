# AGENTS.md

## Overview

This repo is a content-only personal plugin marketplace for Claude Code and Codex. Shared plugin payloads live once under `plugins/<name>/`, while each host gets its own manifest layer and installation surface. The trade-off is deliberate metadata duplication: prompt content stays single-sourced, but version bumps and public descriptions must stay synchronized across marketplace files, plugin manifests, and README surfaces.

## Architecture

```text
.claude-plugin/marketplace.json        Claude Code marketplace index for this repo
README.md                              Repo-level installation and plugin index
plugins/<plugin>/
  .claude-plugin/plugin.json           Claude Code manifest
  .codex-plugin/plugin.json            Codex manifest + interface metadata
  README.md                            Human-facing plugin docs
  skills/<skill>/SKILL.md              Canonical skill prompt with YAML frontmatter
  skills/<skill>/<aux>.md              Step guides or reference material for complex skills
  skills/<skill>/agents/<provider>.yaml Optional provider-specific prompt helpers
  commands/<command>.md                Optional shared slash-command content
```

Host discovery happens through manifest files, not executable entry points. New shipped content belongs under `plugins/<plugin>/`. Root files mainly index, describe, or register plugins. Treat `.claude/settings.local.json` as local workspace configuration, not product content.

## Conventions

- Plugin directories and skill directories use kebab-case, and the directory name matches the exported plugin or skill name.
- Every shipped plugin keeps `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` side by side. Shared metadata fields (`name`, `version`, `description`, `author`, `repository`, `license`) must stay aligned.
- Codex manifests additionally own `skills`, `interface`, `capabilities`, `category`, and `defaultPrompt`. Update them whenever the discoverable surface changes.
- Skill entry files are always `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the prompt body.
- Auxiliary markdown files do not use frontmatter. Use `step-N-<description>.md` for ordered workflows and descriptive kebab-case names such as `layer-reference.md` for reference material.
- Agent-specific helper prompts live under `skills/<skill>/agents/<provider>.yaml`.
- Parent skill files may orchestrate step/reference files, but step files should not reference sibling step files or the parent `SKILL.md` directly. Data flow stays parent -> child.
- When one skill invokes another skill, keep the callee automatable. Avoid wording that forces unnecessary confirmation in sub-skill flows.
- Skill prompts should describe the goal and desired outcome, not over-constrain the implementation technique. Examples are illustrative, not mandatory.
- After renaming steps, adding files, or changing workflow structure, audit cross-skill references, plugin READMEs, and any hard-coded step numbers for rot.
- Root memory files are sibling views of the same project memory. If one changes materially, sync the other.

## How to Add

### Add a New Plugin

1. Create `plugins/<plugin>/`.
2. Add both manifests:
   - `plugins/<plugin>/.claude-plugin/plugin.json`
   - `plugins/<plugin>/.codex-plugin/plugin.json`
3. Add shared payloads as needed: `skills/`, `commands/`, `hooks/`, `agents/`, `.mcp.json`, plus a plugin `README.md`.
4. Register the plugin in `.claude-plugin/marketplace.json`.
5. Update the root `README.md` and the plugin `README.md` if the public installation or usage surface changed.
6. For Codex, remember installation is machine-local through `~/.agents/plugins/marketplace.json`; do not vendor that file into this repo.
7. Commit, push, and tag a release.

### Add or Rename a Skill

1. Create `plugins/<plugin>/skills/<skill>/SKILL.md` with `name` and `description` frontmatter.
2. If the skill is multi-step, add step/reference files beside it instead of embedding the whole workflow in one prompt.
3. Update the plugin README component list and usage examples.
4. If the skill is surfaced in manifest `defaultPrompt` text or other docs, update those references too.
5. If the rename affects cross-skill references, audit every reference for stale skill names or stale step numbers.

### Change Any Shipped Plugin Content

1. Edit the shared Markdown, JSON, or YAML payload.
2. Bump the plugin version in:
   - `plugins/<plugin>/.claude-plugin/plugin.json`
   - `plugins/<plugin>/.codex-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` for that plugin
3. Keep semver discipline:
   - Patch for wording tweaks or small prompt refinements
   - Minor for new skills or commands, or for substantial workflow changes
   - Major for breaking behavior changes
4. Diff the matching README and manifest surfaces before committing.

## Key Entities

| Entity                 | Location                                                 | Role                                                                              |
| ---------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Marketplace entry      | `.claude-plugin/marketplace.json`                        | Registers each plugin name, version, description, and source path for Claude Code |
| Claude plugin manifest | `plugins/<plugin>/.claude-plugin/plugin.json`            | Minimal distributable metadata for Claude Code                                    |
| Codex plugin manifest  | `plugins/<plugin>/.codex-plugin/plugin.json`             | Codex metadata plus UI/interface fields and the relative `skills` root            |
| Skill prompt           | `plugins/<plugin>/skills/<skill>/SKILL.md`               | Canonical task contract for a skill                                               |
| Step/reference file    | `plugins/<plugin>/skills/<skill>/*.md`                   | Breaks complex workflows into ordered steps or reusable reference material        |
| Agent helper config    | `plugins/<plugin>/skills/<skill>/agents/<provider>.yaml` | Optional provider-specific prompt preset layered on top of the shared skill       |

## Commands

### Inspect Structure

```bash
find plugins -type f | sort
rg --files plugins
```

### Check Version Sync

```bash
rg -n '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json plugins/*/.codex-plugin/plugin.json
```

### Review Docs and Metadata Before Commit

```bash
git diff -- README.md .claude-plugin/marketplace.json plugins
git status --short
```

### Validate by Installation

```text
Claude Code:
  /plugin marketplace add yeluyang/skills
  /plugin install <plugin>@yeluyang-skills

Codex:
  clone this repo to ~/src/github.com/yeluyang/skills
  register plugins from ~/.agents/plugins/marketplace.json
```

There is no in-repo build, test, lint, or CI pipeline today. Validation is structural review plus install smoke tests in Claude Code or Codex.
