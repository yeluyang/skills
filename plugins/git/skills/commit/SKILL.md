---
name: commit
description: Generate a commit message via `$git:message` and create the Git commit non-interactively. Use when Codex needs to commit either the already-staged changes or the full current working tree relative to `HEAD`, while preserving a review-friendly message and footer handling. Trigger for requests such as `$git:commit` or `$git:commit HEAD`, especially when the user wants safe staging behavior and a clean commit created from the drafted message.
---

# Create Git Commits

## Overview

Prepare the correct staged change set, delegate message drafting to `$git:message`, and create the commit with that exact message.
Handle commit creation only after the message is fully resolved. Do not open an editor or rely on interactive Git flows.

## Workflow

### 1. Resolve the supported scope

This skill supports only two scope modes:

- `$git:commit`
  Start by treating the commit target as the current staged set only.
  If the staged diff is non-empty, keep the conservative staged-only behavior and draft the message by invoking `$git:message staged`.
  If the staged diff is empty, interpret the no-argument call as an implicit request to commit the full working tree and promote the flow to `HEAD` mode automatically.
- `$git:commit HEAD`
  Stage the full working tree first with `git add -A`.
  After staging, verify that no unstaged tracked changes or untracked files remain.
  Then draft the message by invoking `$git:message staged`.

This skill intentionally does not support commit-range scopes such as `$git:commit <commit>`.
If the user needs a squash-style message for a historical range, route them to `$git:message <commit>` instead of guessing.

### 2. Prepare the commit set safely

For `$git:commit`:

- Inspect the index with `git diff --staged`.
- If the staged diff is non-empty, keep the commit scoped to the index and ignore unstaged changes.
- If the staged diff is empty, do not stop immediately. Promote the workflow to the same preparation path as `$git:commit HEAD`, because the user may have omitted `HEAD` for convenience.

For `$git:commit HEAD`:

- Run `git add -A`.
- Verify that the working tree is now fully staged before proceeding.
- Prefer `git status --porcelain` for verification and confirm that no entry has an unstaged-worktree marker and no untracked entry remains.
- If verification fails, stop and explain what is still outside the commit instead of committing a partial result.
- If the staged diff is empty after `git add -A`, stop and say there is nothing to commit.

Treat the implicit fallback as a user-experience optimization, not as a redefinition of explicit scope. Apply it only to `$git:commit` with no argument.

### 3. Pass through message-related inputs

Forward all message-shaping inputs to `$git:message` unchanged, especially:

- the resolved scope, which is always `staged` once the commit set is ready
- any user-provided co-author information
- any user preference that affects footer handling

Do not re-interpret the diff independently when `$git:message` is the designated message generator. Let `$git:message` own the message content.

If `$git:message` needs follow-up input, such as unresolved co-author details or confirmation about adding the agent as a co-author, resolve that first and commit only after the final message is complete.

### 4. Create the commit non-interactively

Use the message returned by `$git:message` exactly as the commit message text.

Preserve multi-line bodies and footers safely:

- Write the final message to a temporary file.
- Run `git commit --file <tempfile>`.
- Remove the temporary file after the command completes.

Do not use `git commit` without a file when the message contains a body or footer.
Do not launch the user's editor.
Do not rewrite the message unless a purely mechanical correction is required before committing.

### 5. Handle failure modes explicitly

Stop instead of improvising when any of these occur:

- there is no commit-worthy staged diff
- `git add -A` does not fully capture the working tree in `HEAD` mode
- `$git:message` cannot finalize the footer data
- `git commit` fails because of hooks, conflicts, or repository state

Report the concrete blocker and the command or state that caused it.

### 6. Report the result

After a successful commit:

- report the new commit hash via `git rev-parse --short HEAD`
- report the committed subject line via `git log -1 --format=%s`
- mention whether the commit came from staged-only mode or `HEAD` mode
- if no-argument invocation auto-promoted to `HEAD` mode, mention that briefly so the user can see which path was taken

Keep the summary short. The commit itself is the primary artifact.
