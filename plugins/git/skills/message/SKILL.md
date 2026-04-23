---
name: git:message
description: Generate a git-log-review-friendly commit message from repository changes. Use when you need to inspect staged changes, the current working tree relative to `HEAD`, or a commit-to-working-tree range and draft only the commit message text in a conventional-commit-style format, without creating the commit. Trigger for requests such as the skill `git:message`, the skill `git:message` with `staged`, the skill `git:message` with `HEAD`, or the skill `git:message` with `<commit>`, especially when the user wants a squash-ready summary of the most important changes.
---

# Generate Commit Messages

## Overview

Inspect the relevant Git diff, infer the primary intent behind the change, and draft a commit message that is easy to scan in `git log`.
Generate the message only. Do not run `git commit`, `git merge`, `git rebase`, or any other history-changing command unless the invoking context explicitly requests it.

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

Work backwards from the diff by answering three ordered questions. Question 1 is a factual property of the repository; questions 2 and 3 are judgments about the specific change.

1. **What does this repository actually deliver?** Decide what counts as the repository's product surface before classifying anything. The same diff can imply very different intents depending on this answer.
   - Code-first repositories deliver executable artifacts (binaries, libraries, services, apps).
   - Content-first repositories deliver Markdown, prompts, schemas, configuration bundles, design docs, or similar content as the product itself.
   - Infrastructure-first repositories deliver Terraform modules, Helm charts, CI pipelines, or other operational assets.
2. **What is the user's primary intent?**
3. **Which secondary changes materially affect review and deserve to be highlighted?**

#### 3a. Classify the primary intent via a decision tree

First gate: does the change modify the repository's delivered product surface, as defined by question 1?

- **Yes — walk the product-affecting chain in order and take the first matching type. The chain is ordered by prominence to a reviewer, so stronger intents pre-empt weaker ones; `refactor` is the catch-all for everything on the product surface that none of the earlier types claim.**
  1. `feat`: does the change add user-visible capability to the product surface?
     - Yes: use `feat`.
     - No: continue.
  2. `fix`: does the change correct a defect in the product surface?
     - Yes: use `fix`.
     - No: continue.
  3. `perf`: does the change improve performance characteristics of the product surface without changing external behavior?
     - Yes: use `perf`.
     - No: continue.
  4. `style`: is the change limited to formatting or whitespace on the product surface, with no behavior change?
     - Yes: use `style`.
     - No: continue.
  5. `test`: is the change limited to tests or test fixtures that ship with the product surface?
     - Yes: use `test`.
     - No: continue.
  6. `refactor`: fallback — the change restructures the product surface with no external behavior change and none of the above fits. Use `refactor`.
- **No — the change touches only supporting material. Pick the one that matches the nature of that material:**
  - `docs`: changes documentation, comments, or prose that supports but is not itself the delivered product.
  - `chore`: changes build, CI, metadata, tooling, housekeeping, or other ancillary material.

Apply these disambiguating rules carefully:

- In a content-first repository, substantive new end-user content on the delivered surface is a product change, so it maps to `feat` (or `fix`/`refactor`/etc. as appropriate) — not `docs`. `docs` is reserved for supporting prose that is not itself the product.
- In a code-first repository, comment-only edits inside source files are supporting prose, so they map to `docs`, not to a code-change type.
- Do not promote a change to `feat` merely because the diff is large or touches many files. Use `feat` only when the change actually adds user-visible value on the product surface.
- If a single commit legitimately spans both halves of the tree, pick the type that reflects the dominant review concern and surface the rest through the body and secondary-change highlights.

#### 3b. Rank secondary changes by review value

Then rank secondary changes by review value. Highlight only changes that meaningfully help someone scanning `git log`, such as:

- public API shape changes
- module or directory restructures
- schema or contract changes
- cross-cutting cleanup with significant reach
- notable configuration or workflow changes
- migrations, removals, or compatibility-impacting edits

De-emphasize low-signal collateral, such as opportunistic formatting, minor renames, or trivial comment cleanups, unless they are central to the change.

### 4. Draft the commit message

Before writing a single character of the message, lock in one non-negotiable rule: **the entire commit message MUST be written in plain English — title, body, and every footer line.** This overrides every other signal, including the language the user used when invoking the skill, the language of prior commits, branch names, issue titles, code comments, or surrounding documentation, and any inferred locale or team convention. Do NOT emit Chinese, Japanese, Korean, or any other non-English characters anywhere in the output, and do NOT mix languages. Proper nouns, identifiers, file paths, and API names that are already non-English in the codebase stay verbatim, but every surrounding word MUST still be English. If a user request conflicts with this rule, refuse the language switch and produce English anyway.

With that constraint fixed, draft the title, then the body, then the footers in order.

#### 4a. Draft the title

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

#### 4b. Draft the body

Add a body only when it improves clarity.

When a body is needed:

- Separate it from the title with one blank line.
- Explain what changed and why it matters.
- Promote important secondary changes that did not fit in the title.
- Use `-` bullets when describing multiple distinct points.
- Keep the body focused on reviewer value, not a line-by-line transcript of the diff.

#### 4c. Draft footers

This step may append up to two kinds of `Co-Authored-By` footers: human co-authors provided by the user, and the agent's own co-author line. The human co-author section is optional and gated on user input; the agent co-author section is mandatory by default and only suppressed on explicit user request. Always place any human co-author lines before the agent line so attribution reads user-first.

##### 4c-i. Add human co-authors (optional)

Add this section only when it is actually needed, i.e. when the user has provided or implied one or more human co-authors. If no human co-author is in scope, skip this subsection entirely.

For co-authors:

- If the user explicitly provides a co-author, add one `Co-Authored-By: Name <email>` footer per person.
- If the user provides incomplete co-author information, try to recover the missing fields from Git history before asking follow-up questions. Prefer `git log --format='%an <%ae>' --all` and nearby history over guessing.
- If the name or email still cannot be resolved reliably, ask the user for the missing information instead of fabricating it.

##### 4c-ii. Add the agent co-author (mandatory by default)

Always append exactly one agent co-author line. Skip it only when the user explicitly says not to include it.

Format the line exactly as `Co-Authored-By: <Agent Name> <Model Name> <email>`.

The `<email>` MUST belong to the agent vendor, not to the underlying model vendor. Resolve it from the agent brand: Claude-family agents use `noreply@anthropic.com`, Codex uses `noreply@openai.com`, TraeCli uses `noreply@bytedance.com`, and so on. When the agent runs on a third-party or routed model (for example, an OpenRouter-served model inside a ByteDance agent), the email still tracks the hosting agent, not the model provider. If the agent vendor's noreply domain is unknown, ask the user rather than inventing one.

Examples of correct agent footer format:

```text
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>               # baseline: agent and model from the same vendor
Co-Authored-By: Codex GPT-5 <noreply@openai.com>                      # different agent vendor shifts the email domain accordingly
Co-Authored-By: Claude Code Deepseek Reasoner <noreply@anthropic.com> # mixed vendors: model name reflects the actual runtime model, email tracks the agent vendor — never coerce the model name to match the agent's brand
```

Examples of incorrect format — do NOT produce these:

```text
Co-Authored-By: Claude <noreply@anthropic.com>           # missing model name
Co-Authored-By: Claude Opus 4.6 (noreply@anthropic.com)  # parentheses instead of angle brackets
Co-Authored-By: claude-opus-4.6 <noreply@anthropic.com>   # slug instead of display name
Co-Authored-By: Opus 4.6 <noreply@anthropic.com>          # missing agent brand name
```

When both a human co-author and the agent co-author appear, place the human co-author first:

```text
Co-Authored-By: Jane Doe <jane@example.com>
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

### 5. Output contract

Return the proposed commit message text only, formatted exactly as it should be used.

If you still need user input for co-author resolution, ask one concise follow-up question after the draft instead of committing unilaterally.
