---
name: git:message
description: Generate a git-log-review-friendly commit message from repository changes. Use when you need to inspect staged changes, the current working tree relative to `HEAD`, or a commit-to-working-tree range and draft only the commit message text in a conventional-commit-style format, without creating the commit. Trigger for requests such as the skill `git:message`, the skill `git:message` with `staged`, the skill `git:message` with `HEAD`, or the skill `git:message` with `<commit>`, especially when the user wants a squash-ready summary of the most important changes.
---

# Generate Commit Messages

## Overview

Inspect the relevant Git diff, infer the primary intent behind the change, and draft a commit message that is easy to scan in `git log`.
Generate the message only. Do not run `git commit`, `git merge`, `git rebase`, or any other history-changing command.

## Workflow

### 1. Resolve the diff scope

Translate the invocation into one concrete diff target before writing anything:

- the skill `git:message` with `<commit>`: run `git add -N .`, then inspect `git diff <commit>`.
  Treat the scope as every change after that commit up to the current working tree, including staged changes and newly visible untracked files. The user may be preparing a squash; treat that as normal.
- the skill `git:message` with `HEAD`: run `git add -N .`, then inspect `git diff HEAD`.
  Treat the scope as the full current working tree plus staged changes relative to `HEAD`.
- the skill `git:message` with `staged`: inspect `git diff --staged`.
- the skill `git:message` with no argument: start with `git diff --staged` as the conservative default.
  If that staged diff is empty, treat the omission as an implicit request for full-working-tree intent discovery: run `git add -N .`, then inspect `git diff HEAD` instead.

Use `git add -N .` only to expose untracked files in diffs. Do not stage additional content.
Keep explicit scope semantics strict. Apply the fallback only to the no-argument form, not to the skill `git:message` with `staged`.

If the resolved diff is empty, say so instead of inventing a message.

### 2. Build context around the diff

Use lightweight Git context to improve judgment, not to override the diff:

- Inspect file paths, hunk content, and coarse size using the diff itself.
- Check the current branch name with `git branch --show-current` when it may reveal intent.
- Optionally inspect `git status --short` if it helps separate staged from unstaged work.

Treat the branch name as a weak hint only. Ignore vague names or ticket IDs that do not clarify the change.

### 3. Infer the review-relevant story

Work backwards from the diff to answer two questions:

- What is the user's primary intent?
- Which secondary changes materially affect review and deserve to be highlighted?

Classify the primary intent first:

- `feat`: add a user-visible capability
- `fix`: correct a bug
- `perf`: improve performance characteristics
- `style`: change formatting only, with no behavior change
- `test`: change only tests or test fixtures/resources
- `refactor`: change production code but none of the above fits better
- `docs`: change documentation or comments without changing code behavior
- `chore`: change build, CI, metadata, housekeeping, or other non-code and non-doc material

Apply these rules carefully:

- Comment-only edits inside code files count as `docs`, not code changes.
- In documentation-first repositories where the content is the product, substantial new end-user content can still be `feat` even without code changes.
- Do not overuse `feat` just because the diff is large; use it only when the change actually adds user-visible value.

Then rank secondary changes by review value. Highlight only changes that meaningfully help someone scanning `git log`, such as:

- public API shape changes
- module or directory restructures
- schema or contract changes
- cross-cutting cleanup with significant reach
- notable configuration or workflow changes
- migrations, removals, or compatibility-impacting edits

De-emphasize low-signal collateral, such as opportunistic formatting, minor renames, or trivial comment cleanups, unless they are central to the change.

### 4. Draft the title

Use this pattern:

```text
<type>[optional scope]: <description>
```

Choose the optional scope only when it improves searchability or review value. Good scopes are stable module names, subsystems, or meaningful paths. Omit the scope when it would be vague, overly broad, or redundant.

Write the description with high information density:

- Capitalize the subject line.
- Use the imperative mood.
- Do not end the subject line with a period.
- Prefer one precise action over a vague umbrella phrase.
- Do not try to enumerate every side change in the title.

### 5. Draft the body

Add a body only when it improves clarity.

When a body is needed:

- Separate it from the title with one blank line.
- Explain what changed and why it matters.
- Promote important secondary changes that did not fit in the title.
- Use `-` bullets when describing multiple distinct points.
- Keep the body focused on reviewer value, not a line-by-line transcript of the diff.

### 6. Draft footers

Add footer lines only when they are actually needed.

For co-authors:

- If the user explicitly provides a co-author, add one `Co-Authored-By: Name <email>` footer per person.
- If the user provides incomplete co-author information, try to recover the missing fields from Git history before asking follow-up questions. Prefer `git log --format='%an <%ae>' --all` and nearby history over guessing.
- If the name or email still cannot be resolved reliably, ask the user for the missing information instead of fabricating it.

Add yourself as a co-author footer by default.
Skip the co-author footer only when the user explicitly says not to include it.
When included, format the footer exactly as `Co-Authored-By: <Agent Name> <Model Name> <email>`.

Examples of correct agent footer format:

```text
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
Co-Authored-By: Claude Code Deepseek Reasoner <noreply@anthropic.com>
Co-Authored-By: Claude Code GLM 5.1 <noreply@anthropic.com>
Co-Authored-By: Codex GPT-5 <noreply@openai.com>
```

Examples of incorrect format — do NOT produce these:

```text
Co-Authored-By: Claude <noreply@anthropic.com>           # missing model name
Co-Authored-By: Claude Opus 4.6 (noreply@anthropic.com)  # parentheses instead of angle brackets
Co-Authored-By: claude-opus-4.6 <noreply@anthropic.com>   # slug instead of display name
Co-Authored-By: Opus 4.6 <noreply@anthropic.com>          # missing agent brand name
```

When the user co-author and the agent co-author both appear, place the user co-author first:

```text
Co-Authored-By: Jane Doe <jane@example.com>
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

### 7. Output contract

Return the proposed commit message text only, formatted exactly as it should be used.

If you still need user input for co-author resolution, ask one concise follow-up question after the draft instead of performing the commit yourself.
