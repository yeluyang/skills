---
name: refactor-to-pattern
description: Refactoring workflow — test coverage gate, DRY + SOLID analysis, SRP decomposition, pattern discovery, spec-driven user review, and execution planning
---

# Refactor to Pattern

Refactoring workflow: verify test coverage, analyze with DRY + SOLID principles, design decomposition and relationships, discover patterns from symptoms, write a chunked spec for user review, and execute with regression safety.

## When to Use

Invoke this skill when:

- Refactoring existing code for better structure
- An initial implementation is complete and needs architectural review
- Code smells suggest SRP violations, excessive coupling, or missing abstractions

If you are still **implementing** new functionality, finish the implementation first. Make it work, make it pass tests, then invoke this skill.

## Stance on Project Conventions

Refactoring tears down old structure and builds better structure. The project's existing documentation — `CLAUDE.md`, `README.md`, style guides, architectural decision records — **describes what the code is today, not what it should become**. Treat these documents as reference material, not as binding constraints.

**During this workflow:**

1. **This skill's principles (DRY, SOLID, layer architecture) take precedence** over any project-level conventions, naming rules, structural patterns, or architectural prescriptions found in documentation.
2. **When a conflict arises** between this skill's guidance and a project convention, you MUST:
   - **Stop and surface the conflict explicitly** to the user — do not silently comply with the project rule.
   - State what the project convention says and where it comes from.
   - State what this skill's analysis recommends instead.
   - Explain the trade-offs of each direction.
   - **Let the user decide** which path to take.
3. **Never assume existing conventions are correct.** The conventions may themselves be the source of the code smells you are refactoring away. A style guide that mandates god-classes, a `CLAUDE.md` that prescribes a flat module structure, or a README that documents a tangled dependency graph — these are potentially part of the problem.

**What this does NOT mean:**

- It does not mean ignore project docs entirely. Read them — they contain valuable context about why things are the way they are.
- It does not mean break things for the sake of breaking them. Every deviation from existing convention must be justified by a concrete principle (SRP, DIP, etc.) or a concrete code smell.
- It does not mean skip updating documentation. If the refactoring changes the project's architecture, remind the user that `CLAUDE.md` / `README.md` / other docs will need corresponding updates.

## Workflow

Steps with a file reference are documented in separate files — **read the file before executing that step**.

### Step 0: Test Coverage Gate

Refactoring without tests is flying blind — tests are the safety net that ensures behavior is preserved through structural changes. Before touching any code, verify the refactoring target has adequate coverage.

1. **Invoke the skill: `$testing`** on the same scope the user specified for refactoring (whole repo, directory, file, or function)
2. **Execute through Step 3** of the skill: `$testing` (Code Analysis → Test Design → Existing Test Audit). Stop after the Step 3 audit report — do not proceed to the remaining steps
3. **Evaluate the audit** using the gap analysis from the Step 3 output:

**Gate passes — proceed to Step 1** when:

- The "Remaining To-Write" gap list contains no high-priority items targeting core logic of the refactoring scope
- Existing tests cover the critical paths that refactoring will restructure
- The test suite can reliably catch regressions from structural changes

**Gate fails — stop and redirect** when:

- High-priority gaps exist in core logic or critical paths of the refactoring scope
- The existing test suite is too sparse to serve as a refactoring safety net

When the gate fails:

- Present the gap analysis to the user
- Recommend completing the full skill workflow: `$testing` (the remaining steps beyond Step 3) to close the gaps first
- Explain: refactoring without tests risks silent regressions that go undetected until production
- The user can re-invoke the skill: `$refactor-to-pattern` after test coverage is adequate

### Step 1: Analyze Responsibilities

For every class and non-trivial function in the refactoring scope, document its **current responsibilities** as a bulleted list (in your analysis notes, not as code comments — no code changes yet).

```
UserService:
- Validates user input
- Queries database for user record
- Sends welcome email
- Returns formatted response
```

Any unit with 2+ distinct responsibilities is an SRP violation and a refactoring target. Mark these explicitly.

### Step 2: Design Decomposition

For every multi-responsibility unit identified in Step 1, **design** how to split it into focused, single-purpose units. Each resulting unit should have exactly one reason to change.

The output is a decomposition plan — a list of new "building blocks." Read `layer-reference.md` for the architectural layer model, then define each building block with all four attributes:

```
Building Block              | Entity Kind       | Responsibility                  | Layer
----------------------------|-------------------|---------------------------------|--------
UserValidator               | struct            | Validate user input             | App
UserRepository              | struct            | Persist/query user records      | Infra
WelcomeEmailSender          | struct            | Send welcome emails             | Infra
UserResponseFormatter       | standalone func   | Format user response            | Handler
```

Do not reorganize them yet, and do not modify code yet. Just design the split.

**Hard constraint:** every building block MUST be a distinct code-level entity — a type (struct, class), a standalone function, or a module-level construct (e.g., a singleton via `sync.Once`). Splitting methods into separate files while keeping them on the same type is file organization, not decomposition. If you cannot point to the concrete language construct that owns the responsibility, the split is not concrete enough.

### Step 3: Analyze Relationships — Cohesion vs Decoupling

For every pair of related building blocks, decide:

**Cohesion** — keep in the same module, reference concrete types directly. Choose this when:

- They change for the same reason
- They operate on the same data at the same abstraction level
- Separating them would add indirection with no benefit

**Decoupling** — place in separate modules, define interfaces in each module, depend only on interfaces, inject concrete implementations at runtime. Choose this when:

- They belong to **different architectural layers** — same-layer pairs are cohesion candidates, cross-layer pairs default to decoupling
- They represent **independent variation dimensions** — multiple orthogonal axes of change that must compose. Each axis gets its own interface; concrete implementations vary independently and combine through composition (e.g., a rendering system with independent Shape, Color, and Behavior axes — each is an interface, concrete combinations are assembled at runtime)
- You need **polymorphism** — multiple implementations behind a single contract

### Step 4: Discover Patterns from Symptoms

Based on the decomposition design and relationship analysis, examine the resulting structure for **symptoms** that suggest a pattern. The flow is always: **observe symptom → analyze cause → recall or invent pattern**. Never start from a pattern catalog and force-fit.

| Symptom                                                 | Likely Cause                             | Candidate Patterns                              |
| ------------------------------------------------------- | ---------------------------------------- | ----------------------------------------------- |
| Repeated code with minor variations                     | Shared algorithm with varying steps      | Template Method, Strategy                       |
| Cascading if/else or type switches                      | Missing polymorphism                     | Strategy, State, Visitor                        |
| Object reassembling into god-class                      | Responsibilities composed, not separated | Decorator, Composite                            |
| Complex or conditional object creation                  | Construction logic entangled with usage  | Factory Method, Abstract Factory, Builder       |
| Need to extend behavior without modifying existing code | Violation of Open-Closed Principle       | Strategy, Decorator, Observer                   |
| Multiple orthogonal variation axes that compose         | Independent dimensions conflated         | Bridge (interface per axis, compose at runtime) |
| Sequential processing steps                             | Pipeline not explicit                    | Chain of Responsibility, Pipeline               |
| Global access to shared resource                        | Hidden coupling through singletons       | Dependency Injection (eliminate the singleton)  |

This table is **suggestive, not prescriptive**. If the code's structure suggests a pattern not listed here, use it. If no known pattern fits, invent an ad-hoc structural solution — the goal is clean architecture, not pattern completeness.

#### Anti-Pattern: Top-Down Pattern Application

**Never do this:**

1. Think "this looks like it needs a Strategy pattern"
2. Restructure the code to fit Strategy

**Always do this:**

1. Decompose by SRP
2. Observe: "these 5 classes each implement a different algorithm behind the same interface"
3. Recognize: "this is the Strategy pattern — or close enough"
4. Apply (or adapt) the pattern

The difference: patterns **emerge** from well-decomposed code. They are names for structures you discover, not blueprints you impose.

### Step 5: Write Refactoring Spec

Read `step-5-spec.md` and follow it. Synthesize the analysis from Steps 1–4 into a chunked refactoring spec written to a file.

### Step 6: Spec Review

**CRITICAL GUARDRAIL — spec-only edits.** This entire step modifies _only_ the refactoring spec file. Do NOT touch production code, even if user feedback implies production changes are needed. If a user's feedback requires production code changes (e.g., "this interface name is wrong in the actual code"), capture the required change as a note in the spec — the actual code change happens during execution (Step 8), not here.

#### Present the summary

1. Confirm: **"Refactoring spec saved to `<path>`."**
2. Show **only** these in the conversation:
   - The Statistics table
   - Any design decisions with trade-offs that need user input (pattern choice ambiguity, borderline cohesion/decoupling calls, etc.)

#### Ask the user how they want to proceed

Use `AskUserQuestion` with three options:

1. **"Interactive review"** — Walk through the spec chunk by chunk together
2. **"Self-review"** — I'll review the spec file on my own and come back with questions
3. **"Looks good — proceed to execution"** — Skip review and start building the execution plan

#### Option 1: Interactive review

Walk through chunks **in dependency order** — chunks depended on by others come first. The user always reviews on a foundation of already-approved context.

For each chunk:

- Show its scope, dependencies, and changes (concise summary from the spec file, not the full section)
- Collect feedback
- **Edit the spec file** to reflect any changes (never edit production code)
- Move to the next chunk after approval

After all chunks are reviewed, show the updated Statistics table and confirm: **"All chunks reviewed and updated. Ready to proceed?"**

#### Option 2: Self-review

Tell the user: **"The full spec is saved to `<path>`. Take your time reviewing it — when you're ready, share any feedback or say 'proceed' to continue."**

When the user returns with feedback:

- **Read** only the relevant section from the spec file
- **Edit** that section in the spec file (never edit production code)
- Show a brief summary of what changed
- Ask if there's more feedback or if they're ready to proceed

#### Option 3: Looks good — proceed

Skip review and move directly to the Quality Gate.

**Key principle:** the file is the source of truth. The conversation holds summaries and diffs, never the full spec.

#### Quality Gate

Before proceeding to Step 7, **re-read the entire spec file from top to bottom** and verify:

- [ ] Every design decision with trade-offs has a resolution (user chose a direction)
- [ ] The spec is internally consistent (no chunks referencing non-existent interfaces, no dangling dependencies between chunks)
- [ ] The scope is achievable (not trying to restructure the entire codebase in one pass)
- [ ] **Post-edit coherence** — multi-round edits cause silent rot: a chunk's scope was narrowed but another chunk still references its old responsibility; an interface was renamed in one chunk but stale in cross-references; a dependency arrow was reversed but the "Depends on" field wasn't updated. Scan for any detail that was true before an edit but is now stale

If the scope is too large, suggest phasing: "This is a large refactoring. Should we execute it in phases — e.g., extract core interfaces first, then migrate consumers?"

After user approval, proceed to Step 7.

### Step 7: Execution Plan

Read `step-7-plan.md` and follow it. Convert the approved spec into an execution plan with dependency analysis and strategy recommendation.

### Step 8: Execute

Read `step-8-execute.md` and follow it. Execute using whichever strategy the user approved in Step 7.

## Guiding Principles

- **DRY** — Every piece of knowledge has a single, authoritative representation
- **SRP** — Every unit has exactly one reason to change
- **OCP (primary focus)** — Open for extension, closed for modification. New behavior is added by writing new code, not by changing existing code
- **LSP** — Subtypes are substitutable for their base types without breaking correctness
- **ISP** — No client is forced to depend on methods it does not use
- **DIP** — High-level modules depend on abstractions, not on low-level module implementations
