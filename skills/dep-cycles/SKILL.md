# dep-cycles

Detect circular dependencies using DFS on the import graph.

## Trigger

`/dep-cycles` or when user asks about circular dependencies, import cycles, or circular imports.

## Steps

### 1. Build Dependency Graph

Run the dep-graph skill's steps 1–4 to get the adjacency list. If dep-graph was already run in this session, reuse the graph.

### 2. Run DFS Cycle Detection

Use iterative DFS with a visited set and recursion stack:

```
function findCycles(graph):
  visited = {}      # node → "unvisited" | "in-stack" | "done"
  cycles = []

  for each node in graph:
    if visited[node] == "unvisited":
      dfs(node, [], visited, cycles, graph)

function dfs(node, path, visited, cycles, graph):
  visited[node] = "in-stack"
  path.push(node)

  for each neighbor in graph[node]:
    if visited[neighbor] == "in-stack":
      # Found cycle — extract minimal cycle from path
      cycleStart = path.indexOf(neighbor)
      cycle = path.slice(cycleStart) + [neighbor]
      cycles.push(cycle)
    elif visited[neighbor] == "unvisited":
      dfs(neighbor, path, visited, cycles, graph)

  path.pop()
  visited[node] = "done"
```

### 3. Deduplicate Cycles

Normalize each cycle to its lexicographically smallest rotation so `A→B→C→A` and `B→C→A→B` are counted as the same cycle.

### 4. Classify Cycle Severity

| Type | Condition | Severity |
|---|---|---|
| SELF | A imports A | CRITICAL |
| PAIR | A→B→A (2 nodes) | HIGH |
| CHAIN | 3+ nodes | MEDIUM (length ≤ 5) / HIGH (length > 5) |

### 5. Find Import Line Numbers

For each edge in a detected cycle `A → B`:
- Read file A and find the line containing the import of B
- Report as `A:LINE_NUMBER → B`

### 6. Suggest Resolution

For each cycle, suggest which edge to break:
- **Prefer breaking the edge that creates the fewest downstream changes** (break the edge where the imported file has the lowest in-degree from cycle members)
- **Suggest dependency inversion**: if `A → B` creates a cycle, consider extracting a shared interface/type that both A and B import, eliminating the direct dependency

### 7. Output

```
Circular Dependency Analysis
═══════════════════════════════════════

Found 3 circular dependency cycles:

─────────────────────────────────────
[HIGH] PAIR cycle:
  src/store/userStore.ts:12  →  src/utils/api.ts
  src/utils/api.ts:8         →  src/store/userStore.ts

  Resolution: Extract the user-related API types to src/types/user.ts
  and import from there in both files, breaking the cycle.
─────────────────────────────────────

─────────────────────────────────────
[MEDIUM] CHAIN cycle (3 nodes):
  src/components/Header.tsx:5   →  src/hooks/useNav.ts
  src/hooks/useNav.ts:3         →  src/context/NavContext.ts
  src/context/NavContext.ts:15  →  src/components/Header.tsx

  Resolution: Break src/context/NavContext.ts → src/components/Header.tsx
  Consider: NavContext should not import UI components.
  Move the dependency upward: Header imports NavContext, not vice versa.
─────────────────────────────────────

─────────────────────────────────────
[CRITICAL] SELF cycle:
  src/utils/format.ts:3  →  src/utils/format.ts
  (file imports itself)

  Resolution: Remove the self-import on line 3.
─────────────────────────────────────

Summary: 1 critical, 1 high, 1 medium cycle
Recommended fix order: CRITICAL → HIGH → MEDIUM
```

If no cycles found:
```
No circular dependencies detected. Dependency graph is acyclic.
```
