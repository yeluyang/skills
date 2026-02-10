# Step 7: Quick Reference Card

**Goal:** Produce a condensed reference document summarizing everything learned.

Generate this when:

- The user has explored all paths they care about
- The user explicitly requests it
- The user says they are done

## Output Format

````markdown
# <Project Name> — Quick Reference Card

## What Is This?

<1-2 sentence summary>

## Type: <Project Type>

## Tech Stack

| Layer    | Technology   |
| -------- | ------------ |
| Language | Go 1.21      |
| HTTP     | Gin          |
| RPC      | gRPC         |
| DB       | MySQL (GORM) |
| Cache    | Redis        |
| MQ       | Kafka        |
| ...      | ...          |

## Core Entities

| Entity | Description |
| ------ | ----------- |
| User   | ...         |
| Order  | ...         |

## Key Paths

| Path              | What It Does                                               |
| ----------------- | ---------------------------------------------------------- |
| ★ Create Order    | Full checkout: validate → charge → persist → publish event |
| ★ Process Payment | ...                                                        |
| ...               | ...                                                        |

## Directory Map

```
cmd/          — entry points
internal/
  handler/    — HTTP/RPC handlers
  service/    — business logic
  repo/       — data access
  model/      — domain entities
pkg/          — shared libraries
```

## Build & Run

```bash
make build        # compile
make run          # local server
make test         # unit tests
docker-compose up # full stack
```

## Glossary

| Term | Meaning |
| ---- | ------- |
| ...  | ...     |
````
