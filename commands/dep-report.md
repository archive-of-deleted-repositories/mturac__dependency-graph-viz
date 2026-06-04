# /dep-report

**Trigger**: `/dep-report`

**Purpose**: Full architecture health report combining graph metrics, cycle detection, and depth analysis.

**Behavior**:
- Runs all three skills sequentially: `dep-graph` → `dep-cycles` → `dep-depth`
- Builds dependency graph once, reuses it for cycles and depth analysis
- Combines all findings into a single architectural health report
- Assigns an overall health score (0–100)

**Health Score Formula**:
```
score = 100
  - (cycles_critical × 20)
  - (cycles_high × 10)
  - (cycles_medium × 5)
  - (fragile_files × 3)
  - (high_coupling_files × 2)    # files with in-degree > 10
  - (high_responsibility_files)  # files with out-degree > 15
```

**Example output**:
```
Architecture Health Report
═══════════════════════════════════════

Health Score: 67/100  [NEEDS ATTENTION]

Graph Summary:
  Files: 47 | Edges: 134
  Most coupled: src/utils/api.ts (23 importers)
  Most imports: src/pages/Dashboard.tsx (31 imports)

Circular Dependencies: 3 cycles
  1 critical (SELF cycle in src/utils/format.ts)
  1 high (PAIR: userStore ↔ api)
  1 medium (CHAIN: Header → useNav → NavContext → Header)

Depth Analysis:
  Max depth: 11 levels | Fragile files: 4
  Worst: src/utils/math/linalg/core.ts (depth 11)
  Top bottleneck: src/utils/api.ts (score 483)

Priority Actions:
  1. [CRITICAL] Fix self-import in src/utils/format.ts:3
  2. [HIGH] Break PAIR cycle: userStore ↔ api (extract src/types/user.ts)
  3. [HIGH] Reduce src/utils/api.ts imports — it's both a cycle participant and top bottleneck
  4. [MEDIUM] Flatten src/utils/math/ — core.ts is 11 levels deep
  5. [MEDIUM] Split src/pages/Dashboard.tsx — 31 imports suggest too many responsibilities
```
