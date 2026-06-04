# dep-depth

Analyze dependency depth: longest import chains, deep files, and architectural bottlenecks.

## Trigger

`/dep-depth` or when user asks about dependency depth, import chains, deep modules, or bottlenecks.

## Steps

### 1. Build Dependency Graph

Run dep-graph skill steps 1–4 to get the adjacency list. Reuse from session if available.

### 2. Identify Entry Points

Entry points are files with **in-degree = 0** (nothing imports them).
Also explicitly check for common entry file names:
- `index.ts`, `index.tsx`, `main.ts`, `app.ts`, `App.tsx`, `server.ts`

If no entry points found (fully connected graph), use the 5 files with lowest in-degree as starting points.

### 3. Compute Longest Path to Each File (BFS/DFS from entry points)

For each entry point, run BFS to assign a depth level to each reachable file:
- Entry point → depth 0
- Direct import → depth 1
- etc.

When a file is reachable from multiple paths, keep the **maximum depth** (longest path = worst case fragility).

```
depths = {}  # file → max depth

function bfs(entryPoint, graph, depths):
  queue = [(entryPoint, 0)]
  while queue not empty:
    (node, depth) = queue.dequeue()
    if node not in depths or depths[node] < depth:
      depths[node] = depth
    for each child in graph[node]:
      queue.enqueue((child, depth + 1))
```

### 4. Flag Deep Files

Files with depth > 8 are flagged as **FRAGILE** (too many intermediaries between them and the entry point).

Depth thresholds:
| Depth | Assessment |
|---|---|
| 0–3 | Shallow — stable |
| 4–7 | Moderate — acceptable |
| 8–10 | Deep — monitor |
| 11+ | FRAGILE — refactor recommended |

### 5. Find the Longest Dependency Chain

Trace the full path from the entry point to the deepest file:

```
Entry: src/index.ts (depth 0)
  → src/App.tsx (depth 1)
    → src/pages/Dashboard.tsx (depth 2)
      → src/features/analytics/index.ts (depth 3)
        → src/features/analytics/charts/LineChart.tsx (depth 4)
          → src/features/analytics/charts/helpers/scale.ts (depth 5)
            → src/utils/math/interpolate.ts (depth 6)
              → src/utils/math/vector.ts (depth 7)
                → src/utils/math/matrix.ts (depth 8)  ← DEEP
```

### 6. Identify Bottleneck Files

A bottleneck file appears on many different paths through the graph.

Compute **path-through count** for each file:
- For each pair (entry, deep-file): count how many shortest paths pass through node N
- Files with high path-through count = bottlenecks (removing them breaks many paths)

Simple heuristic: bottleneck score = in-degree × out-degree. High product = high bottleneck risk.

Top 5 bottleneck files are reported.

### 7. Output

```
Dependency Depth Analysis
═══════════════════════════════════════

Entry Points: 3 (src/index.ts, src/main.tsx, src/worker.ts)
Max depth found: 11 levels
Files at depth > 8: 4 files (FRAGILE)

Longest Dependency Chain (11 levels):
  src/index.ts
  └─ src/App.tsx
     └─ src/pages/Dashboard.tsx
        └─ src/features/analytics/index.ts
           └─ src/features/analytics/charts/LineChart.tsx
              └─ src/features/analytics/charts/helpers/scale.ts
                 └─ src/utils/math/interpolate.ts
                    └─ src/utils/math/vector.ts
                       └─ src/utils/math/matrix.ts
                          └─ src/utils/math/linalg/lu.ts
                             └─ src/utils/math/linalg/core.ts  ← depth 11, FRAGILE

Fragile Files (depth > 8):
  depth 11  src/utils/math/linalg/core.ts        ← FRAGILE
  depth 10  src/utils/math/linalg/lu.ts          ← FRAGILE
  depth 9   src/utils/math/matrix.ts             ← FRAGILE
  depth 9   src/utils/math/vector.ts             ← FRAGILE

Top 5 Bottleneck Files (breaking these would cause widest damage):
  #1  src/utils/api.ts               — score: 483  (in:21 × out:23)
  #2  src/store/index.ts             — score: 270  (in:18 × out:15)
  #3  src/utils/formatters.ts        — score: 108  (in:12 × out:9)
  #4  src/context/AppContext.tsx     — score: 88   (in:11 × out:8)
  #5  src/hooks/usePermissions.ts    — score: 56   (in:8  × out:7)

Recommendations:
  - src/utils/math/linalg/core.ts is 11 levels deep. Consider moving math utilities
    closer to the features that use them, or creating a separate package.
  - src/utils/api.ts is a major bottleneck. Avoid adding more direct imports from it.
    Consider splitting into domain-specific API modules.
```
