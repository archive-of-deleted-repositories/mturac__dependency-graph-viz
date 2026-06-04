# /dep-depth

**Trigger**: `/dep-depth`

**Purpose**: Analyze dependency depth — longest chains, fragile deeply-nested files, and bottleneck modules.

**Behavior**:
- Invokes the `skills/dep-depth` skill
- Traces longest path from each entry point to every reachable file
- Flags files deeper than 8 levels as FRAGILE
- Computes bottleneck score (in-degree × out-degree) to identify load-bearing modules
- Prints the full longest chain as an ASCII tree

**Example output**:
```
Max depth found: 11 levels
Files at depth > 8: 4 files (FRAGILE)

Longest chain:
  src/index.ts → src/App.tsx → ... → src/utils/math/linalg/core.ts (depth 11)

Top Bottlenecks:
  #1  src/utils/api.ts       score: 483  (in:21 × out:23)
  #2  src/store/index.ts     score: 270  (in:18 × out:15)
```
