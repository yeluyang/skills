# Step 4: Spec Output & User Review

**Goal:** Synthesize all findings from Steps 1-3 into a complete testing spec. Present it to the user for review, iterate on feedback, and get final approval before execution.

## 4.1 Synthesize Testing Spec

Compile the complete spec from all prior steps:

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

## 4.2 Present and Iterate

1. Present the complete spec to the user
2. Ask: **"Does this testing spec look right? Any areas you want to adjust — add tests, remove tests, change priorities, or resolve the flagged decisions?"**
3. If the user has feedback:
   - Apply changes to the spec
   - Re-present the affected sections
   - Ask again until approved
4. When approved, proceed to Step 5

## 4.3 Quality Gate

Before proceeding, verify:

- [ ] Every flagged decision has a resolution (user chose an option)
- [ ] The spec is internally consistent (no tests referencing deleted functions, no gaps)
- [ ] The scope is achievable (not trying to test everything in one pass for a huge codebase)

If the scope is too large, suggest phasing: "This is a large test plan. Should we execute it in phases? Phase 1: leaf unit tests. Phase 2: mid-level. Phase 3: integration."

After user approval, update the progress table and proceed to Step 5.
