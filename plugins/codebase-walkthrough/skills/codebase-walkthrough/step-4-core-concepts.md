# Step 4: Core Concepts

**Goal:** Extract the mental model — the "nouns", "verbs", and design philosophy of this codebase.

## 4.1 Domain Entities

Identify the key "nouns" — the core business objects the system manipulates. Look at:

- Database models / table definitions
- Protobuf / Thrift / GraphQL type definitions
- Struct/class definitions in domain/entity/model directories
- API request/response types

For each entity, note: name, key fields, and relationships to other entities.

## 4.2 Database Schema

If the codebase uses a database, extract or infer the schema:

- Table/collection names and their columns/fields
- Primary keys, foreign keys, indexes
- Relationships (1:1, 1:N, M:N)

Present as a compact table or ER-style ASCII diagram.

## 4.3 Key Abstractions

Identify the important interfaces, traits, or abstract classes that define the extension points and architectural boundaries. These reveal what the original authors considered the "seams" of the system.

## 4.4 Design Philosophy & Style

Infer from the code structure:

| Aspect            | What to look for                                                                          |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Architecture      | Layered? Hexagonal? Clean Architecture? MVC? Event-driven? CQRS? Monolithic? Microkernel? |
| Error Handling    | Error codes? Exceptions? Result types? Sentinel errors?                                   |
| Configuration     | Env vars? Config files? Feature flags? Convention over configuration?                     |
| Naming            | camelCase/snake_case? Prefixes/suffixes? Domain-specific terminology?                     |
| Testing           | Unit-heavy? Integration-heavy? TDD style? Test fixtures/factories?                        |
| Concurrency       | Goroutines? Async/await? Thread pools? Actor model?                                       |
| Code Organization | By feature/domain? By layer/technical concern? Flat? Deep nesting?                        |

## 4.5 Glossary

Produce a glossary of project-specific terms — class names, config keys, domain jargon, abbreviations — that appear repeatedly and may confuse newcomers.

## Output

```markdown
## Core Concepts

### Domain Entities

| Entity | Description             | Key Fields                 | Relationships                   |
| ------ | ----------------------- | -------------------------- | ------------------------------- |
| User   | Registered user account | id, email, role            | has many Orders                 |
| Order  | Purchase order          | id, user_id, status, total | belongs to User, has many Items |
| ...    | ...                     | ...                        | ...                             |

### Database Schema

<compact ER diagram or table listing>

### Key Abstractions

- `UserRepository` (interface) — data access for User entity
- `OrderService` (interface) — order business logic contract
- `EventPublisher` (interface) — async event dispatch
- ...

### Design Philosophy

- **Architecture:** Layered (handler → service → repository)
- **Error Handling:** Custom error codes with wrapped errors
- **Concurrency:** Goroutine-per-request with context cancellation
- ...

### Glossary

| Term | Meaning                           |
| ---- | --------------------------------- |
| PSM  | Product-Service-Module identifier |
| ...  | ...                               |
```
