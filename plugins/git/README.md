# git

Git workflows and repository operations toolkit.

This plugin ships shared Git-oriented skills with a Claude Code manifest, while keeping optional per-agent helpers under `skills/*/agents/*.yaml` for `vercel-labs/skills` users.

## Components

### Skills

- **`git:message`** (`/git:message` in Claude Code; skill name `git:message` in `vercel-labs/skills`) - Generate a git-log-review-friendly commit message from staged changes, the current working tree, or a commit-to-working-tree range. The skill drafts the message only and does not perform the commit.
- **`git:commit`** (`/git:commit` in Claude Code; skill name `git:commit`) - Prepare the correct staged change set, delegate message drafting to `git:message`, and create the commit non-interactively.

## Installation

### Claude Code

```text
/plugin marketplace add yeluyang/skills
/plugin install git@yeluyang-skills
```

Then restart Claude Code.

### vercel-labs/skills

```bash
npx skills add yeluyang/skills --list
npx skills add yeluyang/skills --skill git:message --skill git:commit -a claude-code
```

To target other agents, replace `-a claude-code` with your agent id.

## Usage

- **Draft a commit message from staged changes (Claude Code):** `/git:message` or `/git:message staged`
  The no-argument form starts with staged changes and automatically falls back to `HEAD` when the staged diff is empty.
- **Draft a commit message from the full working tree (Claude Code):** `/git:message HEAD`
- **Draft a squash-ready message from a historical anchor (Claude Code):** `/git:message <commit>`
- **Commit the current staged changes (Claude Code):** `/git:commit`
  The no-argument form prefers the staged set, but automatically promotes to full-working-tree `HEAD` mode when nothing is staged.
- **Stage and commit the full working tree (Claude Code):** `/git:commit HEAD`
- **Other agents via `vercel-labs/skills`:** invoke skill names `git:message` and `git:commit` in your agent workflow.
