# Step 6: Deep Dive

**Goal:** Let the user choose paths, then trace each one end-to-end with full detail.

## 6.1 Path Selection

1. Present the core paths identified in Step 5 with a brief rationale for why each is important.
2. Recommend 1-3 paths to start with. Prioritize:
   - The "happy path" that best represents the system's primary purpose
   - A write path that touches the most core entities
   - A path with interesting complexity (branching, external calls, async)
3. Ask the user: **"Which paths would you like to deep-dive into? I recommend starting with X because..."**

## 6.2 Input / Output

For each selected path:

```markdown
### <Path Name>

**Input:**

- Request type: `CreateOrderRequest`
- Key fields: `user_id`, `items[]`, `payment_method`
- Validation rules: <list>

**Output:**

- Response type: `CreateOrderResponse`
- Key fields: `order_id`, `status`, `total`
- Error cases: <list with error codes>
```

## 6.3 Data Structures & Relationships

List all structs, classes, interfaces, and types involved in this path. Show their relationships:

```markdown
**Types involved:**

- `CreateOrderRequest` (API input) → maps to → `Order` (domain entity)
- `Order` → contains → `[]OrderItem` (1:N)
- `OrderItem` → references → `Product` (lookup)
- `OrderRepository` (interface) → implemented by → `mysqlOrderRepo`
- `PaymentService` (interface) → calls → `payment-gateway` (external)

**Class/struct diagram:**
<ASCII relationship diagram>
```

## 6.4 Data Flow

Trace the request from entry to exit, showing every significant step:

```markdown
**Flow:**

Request
→ [Auth Middleware] — validates JWT, extracts user_id
→ [Rate Limit Middleware] — checks Redis counter
→ OrderHandler.CreateOrder()
→ validate request fields
→ OrderService.Create()
→ ProductService.CheckStock() ──→ [Product DB: SELECT]
→ if out of stock → return ErrOutOfStock
→ Order.Calculate() — compute totals, apply discounts
→ OrderRepo.Save() ──→ [Order DB: INSERT]
→ PaymentService.Charge() ──→ [Payment Gateway: HTTP POST]
→ if payment fails → OrderRepo.UpdateStatus("failed") → return ErrPaymentFailed
→ OrderRepo.UpdateStatus("paid") ──→ [Order DB: UPDATE]
→ EventPublisher.Publish("order.created") ──→ [Kafka]
← return Order
← 201 Created + CreateOrderResponse
```

Highlight:

- **Decision points** (if/switch branches)
- **External calls** (DB, cache, MQ, other services) with the operation type
- **Error handling** paths
- **Async side effects** (events, notifications, background jobs)

## 6.5 Q&A Loop

After completing the deep dive for one path, ask: **"Do you have questions about this path, or shall we move to the next one?"**

- If the user has questions → stay on the current path and answer. Common question types:
  - "Why does it do X before Y?" — explain ordering constraints
  - "What happens if Z fails?" — trace error/retry paths
  - "Where is the code for W?" — point to specific files and line numbers
  - "How does this relate to [other path]?" — explain cross-path interactions
- If the user is satisfied → return to **6.1** to select the next path
- If the user has no more paths to explore → this step is complete
