# Step 4: Spec Output & User Review

**Goal:** Synthesize all findings from Steps 1-3 into a complete testing spec. Present it to the user for review, iterate on feedback, and get final approval before execution.

## 4.1 Synthesize Testing Spec to File

Compile the complete spec from all prior steps and **write it to a file** (e.g., `testing-spec.md` in the project root or a location the user prefers). This keeps the spec out of the conversation context — critical for large codebases.

The file should contain these sections:

### Section 1: Testing Philosophy

Explain the principles that guided your test selection for *this specific codebase*:

- Why you prioritized certain edge cases
- What the code's risk profile looks like (where bugs are most likely)
- How the testing strategy maps to the code's architecture
- Any trade-offs made (e.g., skipping integration tests due to infrastructure complexity)

Keep this to 3-5 bullet points. Be specific to this codebase, not generic testing advice.

### Section 2: Scope Summary

```markdown
**Scope:** <what was analyzed — whole project / directory / specific target>
**Entry points analyzed:** N
**Functions in call graph:** N (M leaf, K mid-level, J top-level)
**Max call depth:** N
**Languages / frameworks:** <from Step 0>
**Test tooling:** <test runner, assertion library, mock framework>
```

### Section 3: Test Plan by Area

Organize by module or package. For each area, show a consolidated view:

```markdown
### <Module/Package Name>

**Existing tests — keep as-is:** N tests
**Existing tests — modify:** N tests
- `test_name`: <specific change>
- ...

**Existing tests — delete:** N tests
- `test_name`: <reason>
- ...

**New tests to write:** N tests
- `function_name` — <test case description> [unit/integration]
- ...
```

### Section 4: Flagged Decisions

Present each flagged decision as a choice for the user:

```markdown
#### Decision 1: <description>

**Context:** <why this is flagged>

**Options:**
1. <option A> — <trade-off>
2. <option B> — <trade-off>
3. <option C> — <trade-off>

**Recommendation:** <your recommendation and why>
```

Common flagged decisions:

- Mock complexity: write complex mocks vs. integration test vs. skip
- Integration test feasibility: set up infrastructure vs. skip
- Test data: generate fixtures vs. ask user for sample data
- External dependencies: mock vs. test contract vs. skip

### Section 5: Statistics

```markdown
| Metric                 | Count   |
| ---------------------- | ------- |
| Existing tests scanned | N       |
| Keep as-is             | N       |
| Modify                 | N       |
| Delete                 | N       |
| New tests to write     | N       |
| **Net change**         | +N / -N |
```

## 4.2 Present Summary and Review

The full spec lives in the file. In the conversation, only show a compact summary.

**CRITICAL GUARDRAIL — spec-only edits.** This entire step modifies *only* the testing spec file. Do NOT touch production code, even if user feedback implies production changes are needed. If a user's feedback requires production code changes (e.g., "this function signature is wrong"), capture the required change as a note in the spec — the actual code change happens during execution (Step 5+), not here.

### Save and present the summary

1. Write the spec file (from 4.1) and confirm: **"Testing spec saved to `<path>`."**
2. Show **only** these in the conversation:
   - The Statistics table (Section 5)
   - Flagged Decisions (Section 4) — these require user input

### Ask the user how they want to proceed

Use `AskUserQuestion` with three options:

1. **"Interactive review"** — Walk through the spec section by section together
2. **"Self-review"** — I'll review the spec file on my own and come back with questions
3. **"Looks good — proceed to execution"** — Skip review and start implementing the test plan

#### Option 1: Interactive review

Walk through the spec in this order, one category at a time:

1. **Tests to delete** — show the list, ask for confirmation or adjustments
2. **Tests to keep as-is** — show the list, ask if any should be modified or removed
3. **Tests to modify** — show each modification, ask for confirmation
4. **New tests to write** — show the list, ask if any should be added, removed, or reprioritized
5. **Flagged decisions** — present each decision, collect the user's choice

For each category:

- Show a concise summary from the spec file (not the full section — pull key items)
- Collect feedback
- **Edit the spec file** to reflect any changes (never edit production code)
- Move to the next category

After all categories are reviewed, show the updated Statistics table and confirm: **"All sections reviewed and updated. Ready to proceed?"**

#### Option 2: Self-review

Tell the user: **"The full spec is saved to `<path>`. Take your time reviewing it — when you're ready, share any feedback or say 'proceed' to continue."**

When the user returns with feedback:

- **Read** only the relevant section from the spec file
- **Edit** that section in the spec file (never edit production code)
- Show a brief summary of what changed
- Ask if there's more feedback or if they're ready to proceed

#### Option 3: Looks good — proceed

Skip the interactive review and move directly to the Quality Gate (4.3).

**Key principle:** the file is the source of truth. The conversation holds summaries and diffs, never the full spec.

## 4.3 Quality Gate

Before proceeding, **re-read the entire spec file from top to bottom** and verify:

- [ ] Every flagged decision has a resolution (user chose an option)
- [ ] The spec is internally consistent (no tests referencing deleted functions, no gaps)
- [ ] The scope is achievable (not trying to test everything in one pass for a huge codebase)
- [ ] **Post-edit coherence** — multi-round edits cause silent rot: a test was removed from one area but still counted in the Statistics table; a function was renamed during review but an older test case still references the old name; a flagged decision was resolved but its implications weren't propagated to affected test entries. Scan for any detail that was true before an edit but is now stale

If the scope is too large, suggest phasing: "This is a large test plan. Should we execute it in phases? Phase 1: leaf unit tests. Phase 2: mid-level. Phase 3: integration."

After user approval, update the progress table and proceed to Step 5.
