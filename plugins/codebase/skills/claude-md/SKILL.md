---
name: codebase:claude-md
description: Generate or update CLAUDE.md — deep-analyze a codebase to extract stable knowledge (architecture, conventions, design philosophy) and volatile operational details (build, test, deploy commands) into agent-optimized project documentation
---

# CLAUDE.md Generator

Analyze a codebase deeply and generate or update a CLAUDE.md that gives AI agents the knowledge they need to work effectively in the project.

## When to Use

Invoke this skill when:

- Starting work on a project that has no CLAUDE.md
- A project's CLAUDE.md is stale or incomplete after significant changes
- Onboarding AI agents to a new codebase
- After a major refactor, migration, or architectural change

## Modes

| Mode          | Trigger                            | Behavior                                                |
| ------------- | ---------------------------------- | ------------------------------------------------------- |
| **Bootstrap** | No CLAUDE.md found at project root | Full generation from scratch                            |
| **Update**    | Existing CLAUDE.md detected        | Regenerate, diff against existing, user reviews changes |

Mode is auto-detected in Step 1.

## Progress Tracker

After completing each step, output a progress table:

```markdown
| Step        | Status    |
| ----------- | --------- |
| 1. Detect   | done      |
| 2. Analyze  | ▶ current |
| 3. Generate | —         |
| 4. Write    | —         |
```

## Guiding Principles

- **Agent-first perspective** — Every sentence answers: "would this help an AI agent write correct, consistent code here?" If not, cut it.
- **Concrete over abstract** — Specific file paths, actual command strings, real code patterns. `pytest -x tests/unit/` beats "run the test suite".
- **Stable knowledge floats up** — Order the document so an agent hitting context limits still gets the most durable, high-value knowledge first.
- **Don't duplicate the code** — Capture _why_ and _how to navigate_, not _what the code does line by line_. If something is obvious from reading the code, don't repeat it.
- **Patterns over inventory** — Describe _how the project is organized_ (the pattern), not _what files currently exist_ (the inventory). An agent can `ls` and `glob`; it cannot infer naming conventions, organizational rules, or where to put new code. A file tree enumerating every file becomes stale on the next commit. A description of the organizational pattern stays valid across many changes.
- **Minimal viable documentation** — Every section earns its place. No boilerplate, no filler, no sections included "just because the template has them".

---

## Step 1: Detect

**Goal:** Determine the project type, check for an existing CLAUDE.md, and select the operating mode.

### 1.1 Check for Existing CLAUDE.md

Search the project root for `CLAUDE.md` (case-sensitive). Also check for common variants: `claude.md`, `.claude/CLAUDE.md`.

- **Found** → Tentatively update mode. Read the existing CLAUDE.md in full and hold it in memory for diffing in Step 4.
- **Not found** → Bootstrap mode.

**Divergence check (update mode only):** After reading the existing CLAUDE.md, assess whether its structure and content are close enough to what this skill would generate. If the existing document is structurally divergent — organized around different sections, contains mostly stale or irrelevant content, or would require rewriting most sections — switch to Bootstrap mode and regenerate fully. Do not waste effort on incremental diffs against a fundamentally different document. Log the decision: `"Existing CLAUDE.md diverges significantly from analysis findings. Switching to Bootstrap mode."`

### 1.2 Scope: One Git Repo = One CLAUDE.md

The unit of work is a single version-controlled project (`.git/` boundary). Rules:

1. **Never** generate one CLAUDE.md that spans multiple independently version-controlled repositories.
2. If the project already has multiple CLAUDE.md files in subdirectories (e.g., monorepo packages each with their own), follow that existing convention — update each one individually.
3. Otherwise, one git project gets one root-level CLAUDE.md, including during bootstrap.

### 1.3 Detect Project Type & Identify Languages

Invoke `/codebase:type` to detect project type, primary languages, frameworks, and package managers.

### 1.4 Output

Log detection results and proceed to Step 2:

```
Mode: Bootstrap / Update
Project type: <type>
Language(s): <languages>
Framework(s): <frameworks>
```

---

## Step 2: Analyze

**Goal:** Deep-scan the codebase across 4 layers to gather raw material. Gather all findings before moving to generation. Use parallel exploration wherever possible.

### Layer 1: Project Identity

**Sources:** README, manifests (package.json, go.mod, Cargo.toml, pyproject.toml, pom.xml, etc.), existing documentation, CONTRIBUTING.md, LICENSE.

**Extract:**

- Project purpose — what problem does this solve, for whom?
- Design philosophy — any stated principles, goals, or non-goals?
- Versioning scheme, release process (if documented)
- Supported platforms / environments

### Layer 2: Structure & Architecture

**Sources:** Top-level directory listing, entry points, import graphs, module boundaries.

**Actions:**

1. Map the top-level directory layout — each directory's role
2. Identify all entry points: `main()` functions, script targets, CLI commands, serverless handlers, route registrations
3. Trace the module dependency graph — which packages import which? Identify layering direction (e.g., handlers → services → repositories)
4. Detect architectural patterns: MVC, clean architecture, hexagonal, monolith, microservice, event-driven, etc.
5. Identify key abstractions: interfaces, traits, abstract classes, protocol definitions

### Layer 3: Conventions & Style

**Sources:** 5-10 representative source files sampled across different modules. Also: linter configs, formatter configs, editorconfig, pre-commit hooks.

**Extract:**

- **Naming:** camelCase vs snake_case vs PascalCase — for files, functions, variables, types, constants
- **File organization:** one class per file? grouped by feature or by layer? test file placement?
- **Error handling:** Result/Option types, exceptions with custom hierarchy, error codes, sentinel errors?
- **Testing patterns:** framework, assertion style, mock/stub approach, fixture patterns, naming convention
- **Import conventions:** absolute vs relative, grouping/ordering rules
- **Code documentation:** docstring style (JSDoc, godoc, rustdoc, etc.)
- **Linter/formatter configs:** active rules that reveal project standards

### Layer 4: Operational Knowledge

**Sources:** Makefile, justfile, package.json scripts, CI config, Dockerfile, docker-compose.yml, .env.example, README "Getting Started".

**Extract:**

- **Build commands:** how to compile / bundle
- **Run commands:** how to start locally, required env vars or config
- **Test commands:** unit, integration, e2e — with prerequisites
- **Lint/format commands:** tools and invocation
- **CI pipeline:** stages, triggers, special requirements
- **Environment setup:** required tools/versions, env vars, config files, database setup, seed data
- **Core data structures:** DB schemas, protobuf/thrift definitions, type definitions — the central entities
- **Critical request paths:** trace 1-2 representative flows end-to-end (handler → business logic → DB → response). Note decision points, external calls, error branches.

All findings feed into Step 3.

---

## Step 3: Generate

**Goal:** Synthesize analysis findings into CLAUDE.md sections — core sections for every project, plus optional sections triggered by discoveries.

### Section Ordering

Stable knowledge first, volatile operational details next, optional sections last:

1. Overview _(core, stable)_
2. Architecture _(core, stable)_
3. Conventions _(core, stable)_
4. How to Add _(core, stable)_
5. Key Entities _(core, semi-stable)_
6. Commands _(core, volatile)_
7. _Optional sections_ _(varies)_

### Core Sections

**1. Overview** — One paragraph. What is this project? What problem does it solve? For whom? What is the design philosophy or key trade-off? Do NOT restate the README — distill the essence.

**2. Architecture** — High-level module map showing how major components relate. ASCII diagram if meaningful. Include: architectural pattern, key abstraction boundaries, direction of dependencies. Level of detail: enough for an agent to know _where to put new code_ and _what depends on what_. Describe the organizational _pattern_ (e.g., "handlers live in `internal/handler/`, one file per resource"), not an exhaustive file inventory. Top-level directory roles with their naming conventions are fine; enumerating every file in each directory is not — that's inventory an agent can discover with `ls`.

**3. Conventions** — Bulleted list of concrete, actionable rules derived from Layer 3 analysis. Only include conventions actually practiced in the codebase — do NOT invent aspirational rules. Example:

```markdown
- Functions use `snake_case`, types use `PascalCase`
- Tests are co-located: `foo.ts` → `foo.test.ts` in the same directory
- Errors use the custom `AppError` class with codes from `src/errors/codes.ts`
- Imports grouped: stdlib → external → internal, separated by blank lines
```

**4. How to Add** — Step-by-step recipes for the most common contribution patterns in this project: adding a new route, model, test, module, config file, etc. This is the highest-value content for agents — it encodes the project's conventions into actionable procedures that stay valid even as individual files come and go. Each recipe should name the pattern, e.g.:

```markdown
**To add a new API endpoint:**

1. Create handler in `internal/handler/<resource>.go`
2. Register route in `internal/routes/routes.go`
3. Add service method to `internal/service/<resource>.go`
4. Add tests: `internal/handler/<resource>_test.go`
```

Only include patterns that the project actually follows. If analysis found fewer than 2 meaningful recipes, skip this section.

**5. Key Entities** — Table of core domain objects / data structures:

```markdown
| Entity | Defined In            | Description                    | Relationships                        |
| ------ | --------------------- | ------------------------------ | ------------------------------------ |
| Order  | `src/models/order.ts` | Purchase order with line items | has many OrderItems, belongs to User |
```

**6. Commands** — Copy-paste ready, grouped by workflow (Development, Testing, Code Quality). Include prerequisites as inline notes.

### Optional Sections

Include only if analysis found relevant material:

| Section                   | Trigger                           | Content                                            |
| ------------------------- | --------------------------------- | -------------------------------------------------- |
| **API Surface**           | HTTP/gRPC/GraphQL routes detected | Endpoint table: method, path, handler, description |
| **CLI Commands**          | CLI command registration detected | Commands with flags and descriptions               |
| **State Management**      | Frontend state library detected   | Store structure, key actions, data flow            |
| **Database & Migrations** | Migration files or ORM config     | Migration commands, schema overview, seeding       |
| **Configuration System**  | Non-trivial config loading        | Config sources, key knobs, defaults                |
| **Deployment**            | Dockerfile, k8s, terraform, CI/CD | How to deploy, environments, infrastructure        |
| **Key Request Paths**     | Critical flows traced in Layer 4  | Step-by-step narrative of representative flows     |

### Self-Evaluation Gate

Before presenting to the user, critically evaluate every section you just generated. For each section, ask three questions:

1. **Volatility** — Will this rot within a few commits? Content that references specific file names, line counts, or current-state inventories rots fast. Content that describes patterns, conventions, and architectural decisions stays valid across many changes. If a section is mostly volatile facts, either rewrite it as stable patterns or cut it.

2. **Agent value** — Does this tell the agent something it cannot easily discover by running `ls`, `glob`, `grep`, or reading 2-3 files? If an agent can figure it out in one tool call, it doesn't belong in CLAUDE.md. CLAUDE.md earns its place by capturing knowledge that requires understanding spread across many files — conventions, architectural intent, non-obvious relationships, "where to put new code" guidance.

3. **Density** — Is this section earning its context window space? A 50-line section that could be replaced by a 2-line pointer ("see `Makefile` for all build targets") is wasting tokens. Prefer terse indexes that point agents to the right files over verbose content that duplicates what's in those files.

**Actions after evaluation:**

- Sections that fail all three: **cut entirely**
- Verbose but valuable sections: **compress to index form** (pattern description + file pointers)
- File trees enumerating every file: **replace with organizational pattern descriptions** (e.g., "one handler file per resource in `internal/handler/`")
- Sections with a mix: **keep the stable core, cut the volatile details**

This gate is not optional. A shorter CLAUDE.md that is 90% stable knowledge beats a longer one that is 50% stale inventory.

All surviving sections feed into Step 4.

---

## Step 4: Write

**Goal:** Assemble and write the final CLAUDE.md.

1. Start with a top-level heading: `# CLAUDE.md`
2. Add sections in order (stable → volatile → optional)
3. Consistent formatting: one blank line between sections, `##` for top sections, `###` for subsections
4. Write to `CLAUDE.md` at the project root — overwrite if it already exists

### Post-Write

Output a brief summary of what was written:

```
CLAUDE.md written with N sections:
- Overview, Architecture, Conventions, ...
Skipped: <any skipped sections, or "none">
```

Remind the user:

- If using chezmoi or similar: the file may need to be re-added or committed
- Re-invoke this skill anytime to update as the project evolves
