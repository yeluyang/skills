# git

Git workflows and repository operations toolkit.

This plugin ships shared Git-oriented skills, with separate plugin metadata for Claude Code and Codex.

## Components

### Skills

- **`message`** (`$git:message` in Codex, `/git:message` in Claude Code) - Generate a git-log-review-friendly commit message from staged changes, the current working tree, or a commit-to-working-tree range. The skill drafts the message only and does not perform the commit.
- **`commit`** (`$git:commit` in Codex, `/git:commit` in Claude Code) - Prepare the correct staged change set, delegate message drafting to `$git:message`, and create the commit non-interactively.

## Installation

### Claude Code

```text
/plugin marketplace add yeluyang/skills
/plugin install git@yeluyang-skills
```

Then restart Claude Code.

### Codex

To make this plugin available from any working directory, clone the repo to a stable path such as `~/src/github.com/yeluyang/skills`, then register it from `~/.agents/plugins/marketplace.json` with:

```json
{
  "name": "yeluyang-skills",
  "interface": {
    "displayName": "yeluyang skills"
  },
  "plugins": [
    {
      "name": "git",
      "source": {
        "source": "local",
        "path": "./src/github.com/yeluyang/skills/plugins/git"
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

This lets Codex discover `git` from any project on that machine without copying the plugin out of the repo.

## Usage

- **Draft a commit message from staged changes:** invoke `$git:message` or `$git:message staged`
  The no-argument form starts with staged changes and automatically falls back to `HEAD` when the staged diff is empty.
- **Draft a commit message from the full working tree:** invoke `$git:message HEAD`
- **Draft a squash-ready message from a historical anchor:** invoke `$git:message <commit>`
- **Commit the current staged changes:** invoke `$git:commit`
  The no-argument form prefers the staged set, but automatically promotes to full-working-tree `HEAD` mode when nothing is staged.
- **Stage and commit the full working tree:** invoke `$git:commit HEAD`
