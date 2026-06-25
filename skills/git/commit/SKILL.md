---
name: git:commit
description: "Draft a review-friendly commit message and create the Git commit non-interactively. Use when you need to commit either the already-staged changes or the full current working tree relative to `HEAD`, while preserving a review-friendly message and footer handling. Trigger for requests such as the skill `git:commit` or the skill `git:commit` with `HEAD`, especially when the user wants safe staging behavior and a clean commit created from the drafted message."
---

# Create Git Commits

## Overview

Prepare the correct staged change set, draft the commit message, and create the commit with that exact message.
Handle commit creation only after the message is fully resolved. Do not open an editor or rely on interactive Git flows.

## Workflow

### 1. Resolve the supported scope

This skill supports only two scope modes:

- the skill `git:commit` without arguments
  Start by treating the commit target as the current staged set only.
  If the staged diff is non-empty, keep the conservative staged-only behavior and proceed with the staged diff.
  If the staged diff is empty, interpret the no-argument call as an implicit request to commit the full working tree and promote the flow to `HEAD` mode automatically.
- the skill `git:commit` with `HEAD`
  Stage the full working tree first with `git add -A`.
  After staging, verify that no unstaged tracked changes or untracked files remain.

This skill intentionally does not support commit-range scopes such as the skill `git:commit` with `<commit>`.

### 2. Prepare the commit set safely

For the skill `git:commit`:

- Inspect the index with `git diff --staged`.
- If the staged diff is non-empty, keep the commit scoped to the index and ignore unstaged changes.
- If the staged diff is empty, do not stop immediately. Promote the workflow to the same preparation path as the skill `git:commit` with `HEAD`, because the user may have omitted `HEAD` for convenience.

For the skill `git:commit` with `HEAD`:

- Run `git add -A`.
- Verify that the working tree is now fully staged before proceeding.
- Prefer `git status --porcelain` for verification and confirm that no entry has an unstaged-worktree marker and no untracked entry remains.
- If verification fails, stop and explain what is still outside the commit instead of committing a partial result.
- If the staged diff is empty after `git add -A`, stop and say there is nothing to commit.

Treat the implicit fallback as a user-experience optimization, not as a redefinition of explicit scope. Apply it only to the skill `git:commit` with no argument.

### 3. Build context around the diff

With the prepared staged set from Step 2, build context around it before writing anything. The relevant diff is whatever Step 2 produced: `git diff --staged` in staged-only mode, or the full working-tree diff relative to `HEAD` after `git add -A` in `HEAD` mode.

Use lightweight Git context to improve judgment, not to override the diff:

- Inspect file paths, hunk content, and coarse size using the diff itself.
- Check the current branch name with `git branch --show-current` when it may reveal intent.
- Optionally inspect `git status --short` if it helps separate staged from unstaged work.

Treat the branch name as a weak hint only. Ignore vague names or ticket IDs that do not clarify the change.

### 4. Infer the review-relevant story

Work backwards from the diff by answering three ordered questions. Question 1 is a factual property of the repository; questions 2 and 3 are judgments about the specific change.

1. **What does this repository actually deliver?** Decide what counts as the repository's product surface before classifying anything. The same diff can imply very different intents depending on this answer.
   - Code-first repositories deliver executable artifacts (binaries, libraries, services, apps).
   - Content-first repositories deliver Markdown, prompts, schemas, configuration bundles, design docs, or similar content as the product itself.
   - Infrastructure-first repositories deliver Terraform modules, Helm charts, CI pipelines, or other operational assets.
2. **What is the user's primary intent?**
3. **Which secondary changes materially affect review and deserve to be highlighted?**

#### 4a. Classify the primary intent via a decision tree

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

#### 4b. Rank secondary changes by review value

Then rank secondary changes by review value. Highlight only changes that meaningfully help someone scanning `git log`, such as:

- public API shape changes
- module or directory restructures
- schema or contract changes
- cross-cutting cleanup with significant reach
- notable configuration or workflow changes
- migrations, removals, or compatibility-impacting edits

De-emphasize low-signal collateral, such as opportunistic formatting, minor renames, or trivial comment cleanups, unless they are central to the change.

### 5. Draft the commit message

Before writing a single character of the message, lock in one non-negotiable rule: **the entire commit message MUST be written in plain English — title, body, and every footer line.** This overrides every other signal, including the language the user used when invoking the skill, the language of prior commits, branch names, issue titles, code comments, or surrounding documentation, and any inferred locale or team convention. Do NOT emit Chinese, Japanese, Korean, or any other non-English characters anywhere in the output, and do NOT mix languages. Proper nouns, identifiers, file paths, and API names that are already non-English in the codebase stay verbatim, but every surrounding word MUST still be English. If a user request conflicts with this rule, refuse the language switch and produce English anyway.

With that constraint fixed, draft the title, then the body, then the footers in order.

The message follows this canonical shape — title, optional body, footers — with one blank line between blocks:

```text
<type>[optional scope]: <subject>

<optional single paragraph>

- <top-level bullet>
  - <nested bullet>
  - <nested bullet>
    - <deeper nesting, depth is unrestricted>
- <top-level bullet>
  - <nested bullet>
  - <nested bullet>

<footer>
```

`type` and `subject` are mandatory; `scope` is optional. The footer is mandatory unless the user explicitly drops it. The body is optional — at most one prose paragraph plus one contiguous list, each independently optional.

Never hard-wrap. Each logical line (subject, paragraph, every bullet, every footer) is one line; let it flow as long as needed — renderers reflow, pre-wrapping only fragments.

The body never holds a second paragraph or a second paragraph-and-list block. If you reach for one, you actually have a list of discrete points — collapse them into a single (possibly nested) list instead. These two shapes are NOT acceptable:

```text
<paragraph>

<paragraph>
```

```text
<paragraph>
- <item>

<paragraph>
- <item>
```

Use `-` for every bullet, indent each nested level two spaces, depth unrestricted.

#### 5a. Draft the title

Use this pattern:

```text
<type>[optional scope]: <subject>
```

Choose the optional scope only when it improves searchability or review value. Good scopes are stable module names, subsystems, or meaningful paths. Omit the scope when it would be vague, overly broad, or redundant.

Write the subject with high information density:

- Capitalize the subject line.
- Use the imperative mood.
- Do not end the subject line with a period.
- Prefer one precise action over a vague umbrella phrase.
- Do not try to enumerate every side change in the title.

#### 5b. Draft the body

Add a body only when it improves clarity.

When a body is needed:

- Separate it from the title with one blank line.
- Explain what changed and why it matters.
- Promote important secondary changes that did not fit in the title.
- Use `-` bullets when describing multiple distinct points.
- Keep the body focused on reviewer value, not a line-by-line transcript of the diff.

#### 5c. Draft footers

This step may append up to two kinds of `Co-Authored-By` footers: human co-authors provided by the user, and the agent's own co-author line. The human co-author section is optional and gated on user input; the agent co-author section is mandatory by default and only suppressed on explicit user request. Always place any human co-author lines before the agent line so attribution reads user-first.

##### 5c-i. Add human co-authors (optional)

Add this section only when it is actually needed, i.e. when the user has provided or implied one or more human co-authors. If no human co-author is in scope, skip this subsection entirely.

For co-authors:

- If the user explicitly provides a co-author, add one `Co-Authored-By: Name <email>` footer per person.
- If the user provides incomplete co-author information, try to recover the missing fields from Git history before asking follow-up questions. Prefer `git log --format='%an <%ae>' --all` and nearby history over guessing.
- If the name or email still cannot be resolved reliably, ask the user for the missing information instead of fabricating it.

##### 5c-ii. Add the agent co-author (mandatory by default)

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

Drafting the message is a subroutine of this skill, not the final output. After the message is fully resolved, proceed to Step 6 to create the commit; do not stop after the draft. If the draft needs follow-up input — unresolved co-author details, or confirmation about adding yourself as a co-author — resolve that first and commit only after the final message is complete.

### 6. Create the commit non-interactively

Use the drafted message exactly as the commit message text.

Preserve multi-line bodies and footers safely:

- Pass the message via a single-quoted heredoc to avoid shell expansion and prevent cleanup failures:
  `git commit -m "$(cat <<'EOF'\n<message>\nEOF\n)"`
- The single quotes around `EOF` disable variable and command substitution inside the heredoc.
- Do not launch the user's editor.
- Do not rewrite the message unless a purely mechanical correction is required before committing.

### 7. Handle failure modes explicitly

Stop instead of improvising when any of these occur:

- there is no commit-worthy staged diff
- `git add -A` does not fully capture the working tree in `HEAD` mode
- the footer data cannot be finalized
- `git commit` fails because of hooks, conflicts, or repository state

Report the concrete blocker and the command or state that caused it.

### 8. Report the result

After a successful commit, show exactly two things:

- which mode produced the commit — staged-only mode or `HEAD` mode (note it briefly if a no-argument invocation auto-promoted to `HEAD` mode)
- the full output of `git log -1`, so the hash, subject, body, and footers are visible together

The commit itself is the primary artifact.
