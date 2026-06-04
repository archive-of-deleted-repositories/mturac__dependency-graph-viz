# /dep-graph

**Trigger**: `/dep-graph`

**Purpose**: Build and display the module dependency graph with coupling metrics.

**Behavior**:
- Invokes the `skills/dep-graph` skill
- Scans all `.ts`, `.tsx`, `.js`, `.jsx`, `.py` files under `src/`
- Extracts and resolves internal imports (relative + path aliases)
- Reports top 10 most-imported files, top 10 files with most imports, entry points, and leaf modules

**Example output**:
```
Dependency Graph Summary
═══════════════════════════════════════

Files analyzed: 47
Total import edges: 134

Top 10 Most-Imported Files (High Coupling Risk):
  #1  src/utils/api.ts              ← imported by 23 files
  #2  src/store/index.ts            ← imported by 18 files
  #3  src/components/ui/Button.tsx  ← imported by 15 files

Top 10 Files with Most Imports:
  #1  src/pages/Dashboard.tsx       → imports 31 files
  #2  src/features/auth/index.ts    → imports 19 files

Entry Points (nothing imports these): 3 files
Leaf Modules (import nothing): 12 files
```
