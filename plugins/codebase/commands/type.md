---
description: Detect project type, primary languages, and frameworks by examining build files, manifests, entry points, and directory structure
---

# Detect Project Type

Examine build files, project manifests, entry points, directory structure, and README to classify the codebase.

## Classification Table

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
| **Data Pipeline / ETL**  | DAG definitions, source-transform-sink structure; scheduler config (Airflow, Spark)                       |
| **ML / AI Project**      | Training loops, model definitions, dataset loaders, inference endpoints                                   |
| **Infrastructure / IaC** | Terraform, Pulumi, CloudFormation, Kubernetes manifests; declarative resource definitions                 |
| **Monorepo**             | Workspace config (nx, turborepo, bazel, lerna); multiple sub-projects with shared packages                |
| **Plugin / Extension**   | Host platform manifest (VS Code `package.json` with `contributes`, browser `manifest.json`)               |
| **Embedded / Firmware**  | Hardware abstraction layers, memory-constrained patterns, RTOS imports                                    |
| **Game**                 | Game loop, scene/entity system, asset pipeline, engine imports                                            |

If the repo is a **monorepo**, identify sub-projects and list them in the output.

## Identify Languages & Frameworks

Quick scan of manifests to identify primary language(s), major frameworks, and package manager(s).

## Evaluate Complexity

After classifying the project type, assess architectural complexity by examining:

- **Codebase size** — approximate lines of code (excluding generated code, vendored dependencies, and test files)
- **Entry points** — how many (single main, multiple servers/cron jobs/CLIs)?
- **External integrations** — databases, caches, message queues, RPC services, third-party APIs?
- **Architectural layering** — flat structure, or distinct layers (presentation, business logic, data access, infrastructure)?
- **Domain modeling** — simple data structs, or rich domain entities with business rules?

| Complexity | Signals                                                                                                                                                                                                                   |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Low**    | Small codebase (< ~1k LoC); single entry point; no or few external integrations; flat directory structure; logic is straightforward data-in/data-out                                                                      |
| **Medium** | Medium codebase (~1k–5k LoC); a distinct presentation/interface layer OR multiple entry points; some external integrations (1–2 databases, an API client); moderate separation of concerns                                |
| **High**   | Large codebase (> ~5k LoC); multiple entry points; multiple external system integrations; distinct architectural layers (handler, service, domain, repository, infrastructure); rich domain modeling; IDL/API definitions |

## Output

Log detection results in the following format:

```
Project type: <type>
Language(s): <languages>
Framework(s): <frameworks>
Package manager(s): <package managers>
Complexity: <Low | Medium | High>
Signals: <brief list of observed structural characteristics that justify the rating>
```
