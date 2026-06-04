# dependency-graph-viz

Build import dependency graph, detect circular dependencies, and analyze deep chains that create brittle architectures.

## Install

```bash
claude plugin install dependency-graph-viz
```

## Commands

| Command | Description |
|---|---|
| `/dep-graph` | Show dependency graph summary with coupling metrics |
| `/dep-cycles` | Detect and show all circular dependencies |
| `/dep-depth` | Show depth analysis and architectural bottlenecks |
| `/dep-report` | Full architecture health report (runs all three) |

## Supported Languages

- TypeScript (`.ts`, `.tsx`)
- JavaScript (`.js`, `.jsx`)
- Python (`.py`)

Resolves: relative imports, dynamic imports, `require()`, path aliases from `tsconfig.json`.

## Example Cycle Detection Output

```
Found 2 circular dependency cycles:

─────────────────────────────────────
[HIGH] PAIR cycle:
  src/store/userStore.ts:12  →  src/utils/api.ts
  src/utils/api.ts:8         →  src/store/userStore.ts

  Resolution: Extract shared types to src/types/user.ts
─────────────────────────────────────

[MEDIUM] CHAIN cycle (3 nodes):
  src/components/Header.tsx:5   →  src/hooks/useNav.ts
  src/hooks/useNav.ts:3         →  src/context/NavContext.ts
  src/context/NavContext.ts:15  →  src/components/Header.tsx

  Resolution: NavContext should not import UI components.
─────────────────────────────────────

Summary: 0 critical, 1 high, 1 medium cycle
```

## Health Score

`/dep-report` produces an overall health score (0–100) combining cycle severity, depth fragility, and coupling risk — with prioritized fix actions.
