# yeluyang-skills

Personal `vercel-labs/skills` repository.

Installable skill payloads live directly under `skills/**/SKILL.md`. This repo intentionally does not ship Claude Code marketplace manifests or plugin wrappers.

## Installation

List available skills:

```bash
npx skills add yeluyang/skills --list
```

Install selected skills for a target agent:

```bash
npx skills add yeluyang/skills --skill codebase:testing --skill git:commit -a codex
```

Replace `codex` with the agent id you want to target.

## Available Skills

| Skill | Location | Description |
| --- | --- | --- |
| `codebase:testing` | `skills/codebase/testing/` | Plan, audit, fill, and prune unit test coverage for production code. |
| `git:commit` | `skills/git/commit/` | Prepare the intended change set, draft a review-friendly commit message, and create the commit non-interactively. |
| `grilling` | `skills/grilling/` | Probe assumptions by exploring broader and narrower interpretations before acting. |

## Repo Layout

```text
README.md
skills/
  <domain>/
    <skill>/
      SKILL.md                         Required skill prompt with YAML frontmatter
      agents/<provider>.yaml           Optional per-agent helper metadata
      <aux>.md                         Optional step guides or reference material
```

`SKILL.md` is the only required file for each skill. Auxiliary Markdown files stay beside the parent `SKILL.md` so relative references remain simple.

Local `skills` CLI runs may create `.agents/` and `skills-lock.json`; both are ignored.

## Adding or Changing Skills

1. Add or edit a skill under `skills/<domain>/<skill>/`.
2. Keep each `SKILL.md` frontmatter limited to `name` and `description`.
3. Put ordered workflow files or references beside the parent `SKILL.md`.
4. Update this README when the public skill list or install examples change.
5. Validate discoverability with:

```bash
npx skills add . --list
```
