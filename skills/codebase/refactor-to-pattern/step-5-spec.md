# Step 5: Write Refactoring Spec

Synthesize the analysis from Steps 1–4 into a structured refactoring spec. **Write it to a file** — keep the full spec out of the conversation context.

**Path:** `refactoring-spec.md` in the project root (or a path the user prefers).

## Chunking the Spec

Break the refactoring into **chunks** — each chunk is one reviewable unit of change. The chunking method is not prescribed; use whatever boundaries are natural for the codebase (feature areas, vertical request flows, modules, layers, API surfaces, event streams, etc.). What matters is that every chunk satisfies these properties:

1. **Independently reviewable** — a reader can judge whether this chunk is correct without holding the details of other chunks in their head
2. **Clear boundary** — explicitly states what is in scope and what is not
3. **Explicit connections** — lists dependencies on other chunks (shared interfaces, data contracts, imported types) so the reader knows where this chunk touches the rest
4. **Right-sized** — small enough to review in one conversation turn; large enough to be a coherent unit of change

## Deduplication

Shared components that appear across multiple chunks are described fully at their **first occurrence**. Every subsequent occurrence uses a brief reference:

```markdown
> See: [<chunk name> → <component>]
```

Do not repeat design details. If a utility, interface, or adapter appears in multiple chunks, the canonical definition lives in one place.

## Spec Structure

```markdown
# Refactoring Spec

## Overview

<What is being refactored, why, key principles applied>

## Chunk: <name>

**Scope:** <what this chunk covers>
**Depends on:** <other chunk names, or "none">
**Changes:**

- **<unit name>** — <single responsibility>
  - Relationships: <cohesion/decoupling decisions from Step 3>
  - Pattern: <if discovered in Step 4>
  - What changes: <what moves, what's new, what's deleted>
- ...

## Chunk: <name>

...

## Statistics

| Metric                               | Count     |
| ------------------------------------ | --------- |
| Chunks                               | N         |
| Modules created / modified / deleted | N / N / N |
| Classes created / modified / deleted | N / N / N |
| Interfaces introduced                | N         |
| Patterns applied                     | N         |
```
