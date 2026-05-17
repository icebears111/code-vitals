---
name: code-sprawl
description: Use when analyzing dependency coupling, finding high fan-in/fan-out files, identifying tightly coupled modules, or planning decoupling and modularization. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Sprawl 🕸️

## What It Does
Analyzes import/require/include graphs to measure coupling. High fan-in (many files import this) = core dependency, risky to change. High fan-out (this file imports many) = god object, needs refactoring.

## Commands

### `/code-sprawl scan`
Scan import/require/include statements across the codebase.

```
═══════════════════════════════════════════
  CODE SPRAWL REPORT
═══════════════════════════════════════════

🕸️  Top Fan-In (most depended on)
  ┌──────────────────────────┬──────────┐
  │ File                     │ Imported │
  │                          │ By       │
  ├──────────────────────────┼──────────┤
  │ src/utils/helpers.ts     │ 47 files │ ← Modifying this breaks 47 callers
  │ src/types/index.ts       │ 38 files │
  │ src/constants.ts         │ 35 files │
  │ src/db/client.ts         │ 29 files │
  │ src/auth/guard.ts        │ 24 files │
  └──────────────────────────┴──────────┘

🕸️  Top Fan-Out (most imports)
  ┌──────────────────────────┬──────────┐
  │ File                     │ Imports  │
  ├──────────────────────────┼──────────┤
  │ src/core/orchestrator.ts │ 23 files │ ← God module
  │ src/api/composite.ts     │ 18 files │
  │ src/services/report.ts   │ 16 files │
  │ src/ui/dashboard.tsx     │ 15 files │
  └──────────────────────────┴──────────┘

⚠️  High Risk: High Fan-In + High Churn
  src/utils/helpers.ts — 47 dependents + 76 edits/year = high risk

🔗  Circular Dependencies
  src/services/order.ts → src/payment/gateway.ts → src/services/order.ts
```

### `/code-sprawl scan --circular`
Only detect circular dependencies. Highlight the cycle path.

### `/code-sprawl scan --risk`
Cross-reference with code-churn: high fan-in + high churn = highest risk files.

### `/code-sprawl scan --graph`
Generate a dependency graph as DOT format or HTML.

## Implementation Notes

1. Search for import/require/include patterns by language:
   - JS/TS: `import`, `require(`, `from `
   - Java: `import `
   - Python: `import `, `from ... import `
   - Go: `import (`
   - Rust: `use `
   - C/C++: `#include `
2. Resolve relative paths to project-relative paths
3. For fan-in: how many files import this file (resolve reverse dependencies)
4. For fan-out: how many imports does this file have
5. Detect circular dependencies by building a directed graph and searching for cycles

## Common Mistakes

- External packages inflate fan-out: exclude node_modules, vendor, site-packages
- Re-exports: `export * from` in JS creates hidden dependencies — trace through
- Wildcard imports: `import java.util.*` — estimate conservatively
- Dynamic imports: `import(variable)` — note them as unknown
