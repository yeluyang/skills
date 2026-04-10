---
name: codebase:memory
description: Generate or update project memory for AI agents — default to AGENTS.md, support agent-specific targets such as CLAUDE.md, and keep sibling memory files synchronized while capturing stable architecture, conventions, and operational knowledge
---

# Project Memory Generator

Analyze a codebase deeply and generate or update its project memory so AI agents can work effectively in the repository.

Default output target: `AGENTS.md`.

If the user names a specific agent, generate that agent's canonical memory file instead. Example: `codebase:memory claude` targets `CLAUDE.md`.

## When to Use

Invoke this skill when:

- Starting work on a project that has no agent memory yet
- A project's memory is stale or incomplete after significant changes
- Onboarding AI agents to a new codebase
- After a major refactor, migration, or architectural change
- A repo needs `AGENTS.md` and `CLAUDE.md` kept aligned

## Target Resolution

Resolve the target file before doing deeper analysis:

1. If the user explicitly names an agent, target that agent's canonical project memory file.
2. If the user does not name an agent, target `AGENTS.md`.
3. Use these built-in mappings unless the repository already proves a different convention:
   - `codex` → `AGENTS.md`
   - `claude` → `CLAUDE.md`
4. For other agents, inspect the repository for existing agent-memory files, agent-specific setup, or documented conventions before inventing a filename.
5. Treat root-level agent-memory files as sibling views of the same project memory. Their shared substance should stay aligned; only agent-specific wording, filenames, or tool notes may differ.

## Modes

| Mode          | Trigger                                                           | Behavior                                                         |
| ------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Bootstrap** | No relevant project-memory file exists                            | Generate the target from scratch                                 |
| **Adapt**     | Target file is missing, but a sibling memory file already exists  | Seed from the sibling file, retarget it, then refresh from code  |
| **Update**    | Target file already exists                                        | Regenerate, diff against existing, user reviews changes          |
| **Sync**      | Multiple sibling memory files exist (`AGENTS.md`, `CLAUDE.md`, …) | Update the requested target and reconcile the other siblings too |

Mode is auto-detected in Step 1. More than one mode may apply; for example, `Adapt + Sync`.

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

- **Agent-first perspective** — Every sentence should help an agent make better decisions in this repo. If not, cut it.
- **One truth, many renderings** — `AGENTS.md`, `CLAUDE.md`, and other sibling memory files may vary in product-specific wording, but they must not disagree about architecture, conventions, or workflows.
- **Concrete over abstract** — Prefer specific file paths, actual command strings, and real code patterns. `pytest -x tests/unit/` beats "run the test suite".
- **Stable knowledge floats up** — Order the document so an agent hitting context limits still gets the most durable, high-value knowledge first.
- **Don't duplicate the code** — Capture _why_ and _how to navigate_, not _what the code does line by line_.
- **Patterns over inventory** — Describe organizational patterns, conventions, and placement rules. Do not dump file trees that rot immediately.
- **Minimal viable documentation** — Every section must earn its context-window cost.

---

## Step 1: Detect

**Goal:** Determine the project-memory target, discover sibling memory files, and select the operating mode.

### 1.1 Discover Existing Project Memory

Search the project root for existing agent-memory files, starting with:

- `AGENTS.md`
- `CLAUDE.md`

Also check obvious local variants already used by the repo, such as `claude.md` or `.claude/CLAUDE.md`.

Classify what you find:

- **Target exists** → Update mode
- **Target missing, sibling exists** → Adapt mode
- **Multiple sibling memory files exist** → Sync mode
- **No relevant memory files** → Bootstrap mode

Read every relevant existing memory file in full. They are inputs to the durability and synchronization pass in Step 3.

### 1.2 Scope: One Git Repo = One Project-Memory Set

The unit of work is a single version-controlled project (`.git/` boundary). Rules:

1. **Never** generate one memory file that spans multiple independently version-controlled repositories.
2. If the project already has multiple memory files in subdirectories (for example, monorepo packages each with their own memory), follow that convention and update each unit independently.
3. Otherwise, one git project gets one root-level project-memory set, even if that set is represented by multiple sibling files such as `AGENTS.md` and `CLAUDE.md`.

### 1.3 Resolve the Requested Agent

If the user included an agent selector, normalize it early and map it to the target filename.

Examples:

- `codebase:memory` → `AGENTS.md`
- `codebase:memory claude` → `CLAUDE.md`
- `codebase:memory codex` → `AGENTS.md`

If the requested agent is unfamiliar, inspect the repo for evidence of its required memory filename or format before deciding. Only ask the user if local evidence remains ambiguous.

### 1.4 Detect Project Type & Identify Languages

Invoke the skill: `$codebase:type` to detect project type, primary languages, frameworks, and package managers.

### 1.5 Output

Log detection results and proceed to Step 2:

```text
Requested agent: <default / agent name>
Target file: <filename>
Sibling memory files: <list or "none">
Mode: <Bootstrap / Adapt / Update / Sync>
Project type: <type>
Language(s): <languages>
Framework(s): <frameworks>
```

---

## Step 2: Analyze

**Goal:** Deep-scan the codebase across 4 layers to gather raw material. Gather all findings before moving to generation. Use parallel exploration wherever possible.

### Layer 1: Project Identity

**Sources:** README, manifests (`package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `pom.xml`, etc.), existing documentation, `CONTRIBUTING.md`, `LICENSE`, existing memory files.

**Extract:**

- Project purpose — what problem does this solve, for whom?
- Design philosophy — stated principles, goals, or non-goals
- Versioning scheme and release process, if documented
- Supported platforms and environments

### Layer 2: Structure & Architecture

**Sources:** Top-level directory listing, entry points, import graphs, module boundaries.

**Actions:**

1. Map the top-level directory layout and each directory's role.
2. Identify all entry points: `main()` functions, script targets, CLI commands, serverless handlers, route registrations.
3. Trace the module dependency graph and layering direction.
4. Detect architectural patterns: MVC, clean architecture, hexagonal, monolith, microservice, event-driven, and so on.
5. Identify key abstractions: interfaces, traits, abstract classes, protocol definitions.

### Layer 3: Conventions & Style

**Sources:** 5-10 representative source files sampled across different modules. Also: linter configs, formatter configs, `.editorconfig`, pre-commit hooks.

**Extract:**

- **Naming:** camelCase vs snake_case vs PascalCase for files, functions, variables, types, constants
- **File organization:** grouped by feature or by layer, test file placement, one-class-per-file patterns
- **Error handling:** result types, exceptions, error codes, sentinel errors
- **Testing patterns:** framework, assertion style, mocks/stubs, fixtures, naming
- **Import conventions:** absolute vs relative, grouping, ordering
- **Code documentation:** JSDoc, godoc, rustdoc, inline comment norms
- **Linter/formatter configs:** active rules that reveal project standards

### Layer 4: Operational Knowledge

**Sources:** `Makefile`, `justfile`, package-manager scripts, CI config, `Dockerfile`, `docker-compose.yml`, `.env.example`, README setup docs.

**Extract:**

- **Build commands:** how to compile or bundle
- **Run commands:** how to start locally, required env vars or config
- **Test commands:** unit, integration, e2e, and prerequisites
- **Lint/format commands:** tools and invocation
- **CI pipeline:** stages, triggers, special requirements
- **Environment setup:** required tools, versions, env vars, config files, database setup, seed data
- **Core data structures:** DB schemas, protobuf/thrift definitions, central types
- **Critical request paths:** trace 1-2 representative flows end to end

All findings feed into Step 3.

---

## Step 3: Generate

**Goal:** Synthesize the analysis into project-memory sections, then align every sibling memory file around the same truth.

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

**1. Overview** — One paragraph. Distill what the project is, what problem it solves, and the key design trade-off. Do not restate the README.

**2. Architecture** — High-level module map showing how major components relate. ASCII diagram if useful. Explain where new code belongs and what depends on what.

**3. Conventions** — Concrete, actionable rules derived from the actual codebase. Do not invent aspirational rules.

**4. How to Add** — Step-by-step recipes for the most common contribution patterns in this project: route, model, module, config, test, migration, and so on. Skip the section if analysis found fewer than 2 meaningful recipes.

**5. Key Entities** — Table of core domain objects, schemas, or central types.

**6. Commands** — Copy-paste-ready commands grouped by workflow, with prerequisites inline.

### Optional Sections

Include only if analysis found relevant material:

| Section                   | Trigger                           | Content                                            |
| ------------------------- | --------------------------------- | -------------------------------------------------- |
| **API Surface**           | HTTP/gRPC/GraphQL routes detected | Endpoint table: method, path, handler, description |
| **CLI Commands**          | CLI command registration detected | Commands with flags and descriptions               |
| **State Management**      | Frontend state library detected   | Store structure, key actions, data flow            |
| **Database & Migrations** | Migration files or ORM config     | Migration commands, schema overview, seeding       |
| **Configuration System**  | Non-trivial config loading        | Config sources, key knobs, defaults                |
| **Deployment**            | Docker, k8s, Terraform, CI/CD     | How to deploy, environments, infrastructure        |
| **Key Request Paths**     | Critical flows traced in Layer 4  | Step-by-step narrative of representative flows     |

### Durability Assessment

Compare each piece of existing memory content against the analysis findings. Evaluate by one criterion:

**Is this content still accurate and does it still guide agents effectively in the current codebase?**

Classify each piece as:

- **Durable** — still true and still helpful
- **Stale** — wrong, misleading, or no longer practiced
- **Uncertain** — cannot be assessed confidently from codebase evidence alone

Origin does not matter. Hand-written guidance can rot. Generated guidance can remain durable.

### Cross-Agent Synchronization Rules

When multiple sibling memory files exist, or when the target must be created from a sibling file:

1. Build one canonical content model from Step 2 and the durable existing content.
2. Render that model into every relevant sibling memory file in the same pass.
3. Keep these sections semantically aligned across siblings:
   - Overview
   - Architecture
   - Conventions
   - How to Add
   - Key Entities
   - Commands
4. Allow differences only where the agent genuinely requires them:
   - Filename and top-level heading
   - Product-specific invocation syntax
   - Agent-specific tooling constraints or workflow notes
5. If only one sibling exists and the requested target is missing:
   - `CLAUDE.md` exists but `AGENTS.md` is missing → copy `CLAUDE.md` to `AGENTS.md`, then retarget and refresh it
   - `AGENTS.md` exists but `CLAUDE.md` is missing → copy `AGENTS.md` to `CLAUDE.md`, then retarget and refresh it
6. Do not let siblings drift into contradictory architecture, convention, or command guidance.

### Self-Evaluation Gate

Before presenting to the user, critically evaluate every section:

1. **Volatility** — rewrite or cut content that will rot within a few commits.
2. **Agent value** — if an agent can discover it with one trivial tool call, it probably does not belong here.
3. **Density** — compress verbose sections into stable, high-signal guidance.

Actions after evaluation:

- Sections that fail all three: **cut entirely**
- Verbose but valuable sections: **compress to index form**
- File trees enumerating every file: **replace with organizational pattern descriptions**
- Mixed sections: **keep the stable core, cut the volatile details**

All surviving sections feed into Step 4.

---

## Step 4: Write

**Goal:** Assemble and write the final project-memory files.

### Bootstrap Mode

1. Start with a top-level heading matching the target filename, for example `# AGENTS.md` or `# CLAUDE.md`.
2. Add sections in order: stable, then volatile, then optional.
3. Use consistent formatting: one blank line between sections, `##` for top sections, `###` for subsections.
4. Write the target file at the project root.

### Adapt Mode

1. Copy the best existing sibling memory file into the missing target file.
2. Adjust only the agent-specific framing first: heading, filename references, invocation syntax, tooling notes.
3. Re-run the durability and synchronization pass so the new target is not a blind copy; it must reflect the current codebase.

### Update / Sync Mode

1. Assemble the new canonical project-memory content: generated sections plus durable existing content.
2. Place durable existing content in its most natural section. Preserve its location when reasonable.
3. Diff each file that will change. Present changes to the user as:
   - **Added** — new content from fresh analysis
   - **Modified** — existing content updated to reflect current state
   - **Removed** — existing content assessed as stale, with reason
   - **Preserved** — durable existing content carried forward
   - **Uncertain** — content requiring a user decision
4. Write after user confirms when an existing file would change.

### Post-Write

Output a brief summary:

```text
Project memory written:
- Target: <filename>
- Siblings updated: <list or "none">
- Sections: <list>
- Preserved from existing: <list or "none">
- Removed as stale: <list with reasons, or "none">
- Skipped: <list or "none">
```

Remind the user:

- If using chezmoi or similar, the file may need to be re-added or committed
- Re-invoke this skill anytime to refresh the project memory as the codebase evolves
