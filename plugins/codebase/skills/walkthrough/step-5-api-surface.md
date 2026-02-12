# Step 5: API Surface & Request Paths

**Goal:** Enumerate all external-facing APIs and identify which are core paths.

## 5.1 Enumerate APIs

Search for route/handler registrations, RPC service definitions, CLI command registrations, event handlers, or cron job definitions — whatever constitutes the "entry surface" for this project type.

For each API/path, record:

- **Endpoint** (route, RPC method, command name, event topic)
- **Method** (GET/POST/PUT/DELETE for HTTP; Unary/Stream for RPC)
- **Brief description** of what it does

## 5.2 Group APIs

Organize by functional domain:

- **CRUD operations** on core entities
- **Business workflows** (multi-step, cross-entity operations)
- **Integration endpoints** (webhooks, callbacks, sync points)
- **Admin / internal** operations
- **Health / observability** endpoints

## 5.3 Highlight Core Paths

Mark paths as **core** if they:

- Operate on core domain entities from Step 4
- Perform database writes (mutations are usually more critical than reads)
- Involve complex business logic or multi-step workflows
- Are called by other services (fan-in indicates importance)
- Have dedicated test coverage (signals developer priority)

## Output

```markdown
## API Surface

### Core Paths (★)

| Endpoint                    | Method | Description                                 |
| --------------------------- | ------ | ------------------------------------------- |
| ★ `/api/v1/orders`          | POST   | Create a new order (full checkout workflow) |
| ★ `/api/v1/orders/{id}/pay` | POST   | Process payment for an order                |
| ...                         | ...    | ...                                         |

### CRUD

| Endpoint             | Method | Description                |
| -------------------- | ------ | -------------------------- |
| `/api/v1/users`      | GET    | List users with pagination |
| `/api/v1/users/{id}` | GET    | Get user detail            |
| ...                  | ...    | ...                        |

### Business Workflows

| ... |

### Integration

| ... |

### Admin & Observability

| ... |
```
