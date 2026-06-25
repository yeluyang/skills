# AGENTS.md

## Overview

This repo is a content-only `vercel-labs/skills` repository. Prompt content lives directly under `skills/**/SKILL.md`.

There are no Claude Code marketplace manifests, plugin wrappers, or version metadata to maintain.

## Architecture

```text
README.md
skills/
  <domain>/
    <skill>/
      SKILL.md                         Canonical skill prompt with YAML frontmatter
      agents/<provider>.yaml           Optional provider-specific helper metadata
      <aux>.md                         Optional step guides or reference material
```

Discovery happens through `SKILL.md` metadata. New shipped content belongs under `skills/`. Treat `.claude/settings.local.json`, `.agents/`, and `skills-lock.json` as local workspace state, not product content.

## Conventions

- Skill directories use kebab-case path segments under `skills/<domain>/<skill>/`.
- Exported skill names in `SKILL.md` may use a namespace such as `codebase:type` or `git:commit`.
- Skill entry files are always `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the prompt body.
- Auxiliary Markdown files do not use frontmatter. Use `step-N-<description>.md` for ordered workflows and descriptive kebab-case names such as `layer-reference.md` for reference material.
- Agent-specific helper prompts live under `skills/<domain>/<skill>/agents/<provider>.yaml`.
- Parent skill files may orchestrate step/reference files, but step files should not reference sibling step files or the parent `SKILL.md` directly. Data flow stays parent -> child.
- When one skill invokes another skill, keep the callee automatable. Avoid wording that forces unnecessary confirmation in sub-skill flows.
- Skill prompts should describe the goal and desired outcome, not over-constrain the implementation technique. Examples are illustrative, not mandatory.
- After renaming steps, adding files, or changing workflow structure, audit cross-skill references, the root README, and any hard-coded step numbers for rot.
- Root memory files are sibling views of the same project memory. If one changes materially, sync the other.

## How to Add

### Add or Rename a Skill

1. Create `skills/<domain>/<skill>/SKILL.md` with `name` and `description` frontmatter.
2. If the skill is multi-step, add step/reference files beside it instead of embedding the whole workflow in one prompt.
3. Update the root README when the public skill list, install snippets, or usage surface changes.
4. If the rename affects cross-skill references, audit every reference for stale skill names or stale step numbers.
5. Validate discoverability with `npx skills add . --list`.

### Change Any Shipped Skill Content

1. Edit the shared Markdown or YAML payload under `skills/`.
2. Keep related helper metadata in `agents/*.yaml` aligned with the parent `SKILL.md`.
3. Diff the matching README and skill surfaces before committing.

## Key Entities

| Entity | Location | Role |
| --- | --- | --- |
| Skill prompt | `skills/<domain>/<skill>/SKILL.md` | Canonical task contract for a skill |
| Step/reference file | `skills/<domain>/<skill>/*.md` | Breaks complex workflows into ordered steps or reusable reference material |
| Agent helper config | `skills/<domain>/<skill>/agents/<provider>.yaml` | Optional provider-specific prompt preset layered on top of the shared skill |

## Commands

### Inspect Structure

```bash
find skills -type f | sort
rg --files skills
```

### Review Before Commit

```bash
git diff -- README.md AGENTS.md CLAUDE.md skills
git status --short
```

### Validate Installation

```bash
npx skills add . --list
npx skills add . --skill <skill> -a <agent>
```

There is no in-repo build, test, lint, or CI pipeline today. Validation is structural review plus `vercel-labs/skills` install smoke tests.
