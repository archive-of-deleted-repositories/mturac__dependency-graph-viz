# /dep-cycles

**Trigger**: `/dep-cycles`

**Purpose**: Detect circular dependencies and show exact import chains with file:line references.

**Behavior**:
- Invokes the `skills/dep-cycles` skill
- Runs DFS cycle detection on the full dependency graph
- Classifies each cycle: SELF / PAIR / CHAIN with severity CRITICAL / HIGH / MEDIUM
- Shows exact import line numbers for each edge in the cycle
- Suggests which edge to break and how (dependency inversion pattern)

**Example output**:
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

  Resolution: Break NavContext → Header. NavContext should not import UI components.
─────────────────────────────────────

Summary: 0 critical, 1 high, 1 medium cycle
```
