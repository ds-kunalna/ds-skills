---
description: Find and categorize services using postgres-lro into LRO services and worker services
---

# List LRO Services

This skill analyzes the codebase to find all services using `libgo/postgres-lro` and categorizes them into LRO services (that create operations) and worker services (that execute operations).

## Steps

1. Search for all files importing `libgo/postgres-lro`:
   ```
   Grep(pattern: "libgo/postgres-lro", output_mode: "files_with_matches")
   ```

2. Find LRO services by searching for services that use `lro.Handler`:
   ```
   Grep(pattern: "lro\.(Handler|NewHandler)", output_mode: "content", glob: "**/service/handler.go", -n: true)
   ```

3. Extract unique service names from both searches by looking at the directory structure under `services/`

4. Categorize services:
   - **LRO Services**: Services that have `handler.go` files using `lro.Handler` interface
   - **Worker Services**: Services ending in `-worker` that poll and execute operations

5. Present results in two categorized lists with clickable markdown links to service directories

## Output Format

Provide a summary with:
- Total count of services using postgres-lro
- List of LRO services (with links)
- List of worker services (with links)
- Brief explanation of the difference between the two types
