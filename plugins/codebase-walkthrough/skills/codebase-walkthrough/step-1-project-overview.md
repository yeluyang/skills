# Step 1: Project Overview

**Goal:** Identify what this codebase is and what it does at the highest level.

## 1.1 Detect Project Type

Examine build files, project manifests, entry points, directory structure, and README to classify the codebase. Confirm with the user.

| Type                     | Signals                                                                                                   |
| ------------------------ | --------------------------------------------------------------------------------------------------------- |
| **API Protocol / IDL**   | Repo is mostly `.proto`, `.thrift`, `.graphql`, or OpenAPI specs; no application runtime code             |
| **Documentation Site**   | Static site generator config (docusaurus, mkdocs, hugo); content is primarily markdown                    |
| **CLI (single-run)**     | Single entry point, argument parser, runs-to-completion; no listener or server loop                       |
| **CLI (daemon)**         | Entry point with long-running loop, file watcher, or socket listener                                      |
| **Web Frontend**         | Component framework (React, Vue, Angular, Svelte); bundler config; `src/pages` or `src/routes`            |
| **Mobile App**           | Platform manifests (AndroidManifest.xml, Info.plist); or cross-platform framework (Flutter, React Native) |
| **Desktop App**          | Electron/Tauri config, native GUI framework imports, window management code                               |
| **Network Service**      | Server bootstrap, route/handler registration, middleware pipeline, database connections                   |
| **SDK / Library**        | Exports a public API; no `main` entry point; published to a package registry                              |
| **Data Pipeline / ETL**  | DAG definitions, source→transform→sink structure; scheduler config (Airflow, Spark)                       |
| **ML / AI Project**      | Training loops, model definitions, dataset loaders, inference endpoints                                   |
| **Infrastructure / IaC** | Terraform, Pulumi, CloudFormation, Kubernetes manifests; declarative resource definitions                 |
| **Monorepo**             | Workspace config (nx, turborepo, bazel, lerna); multiple sub-projects with shared packages                |
| **Plugin / Extension**   | Host platform manifest (VS Code `package.json` with `contributes`, browser `manifest.json`)               |
| **Embedded / Firmware**  | Hardware abstraction layers, memory-constrained patterns, RTOS imports                                    |
| **Game**                 | Game loop, scene/entity system, asset pipeline, engine imports                                            |

If the repo is a **monorepo**, identify sub-projects and ask the user which one to analyze first. Apply the full workflow to that sub-project.

## 1.2 Identify Entry Points

Find all entry points: `main()` functions, `package.json` scripts, `Makefile` targets, CLI commands, serverless handlers, etc. List them with one-line descriptions.

## 1.3 Output

Produce a structured overview:

```markdown
## Project Overview

- **Name:** <project name>
- **Type:** <detected type>
- **Language(s):** <primary and secondary languages>
- **Entry Points:**
  - `cmd/server/main.go` — starts the HTTP/gRPC server
  - `cmd/migrate/main.go` — runs database migrations
- **Top-Level Structure:**
  - `cmd/` — entry points
  - `internal/` — application code
  - `pkg/` — shared libraries
  - ...
- **Summary:** <2-3 sentence description of what this project does>
```

## 1.4 Adaptive Routing

After producing the overview, consult the adaptive routing table in `SKILL.md` to determine which steps apply to this project type (steps 1-7). Tell the user which steps will be executed and why, then proceed.
