---
name: code-chaos
description: Use when measuring code complexity, finding the most convoluted functions, detecting nesting hell, auditing cyclomatic complexity, or prioritizing simplification targets. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Chaos 🌪️

## What It Does
Measures code complexity at the function level. Cyclomatic complexity, nesting depth, line length, and parameter count surface the most chaotic code that needs simplification.

## Commands

### `/code-chaos scan`
Analyze all source files and rank functions by composite chaos score.

```
═══════════════════════════════════════════
  CODE CHAOS REPORT
═══════════════════════════════════════════

🌪️  Top 10 Most Chaotic Functions
  ┌────────────────────────────────┬───────┬──────┬──────┬─────────┐
  │ Function                       │ Score│ Branc│ Depth│ Lines   │
  ├────────────────────────────────┼───────┼──────┼──────┼─────────┤
  │ OrderController.checkout()     │ 92   │ 34   │ 7    │ 312     │ ← 🔴
  │ PaymentGateway.process()       │ 87   │ 28   │ 6    │ 245     │ ← 🔴
  │ MigrationRunner.upgrade()      │ 78   │ 22   │ 8    │ 198     │ ← 🔴
  │ UIRenderer.composeLayout()     │ 65   │ 18   │ 5    │ 167     │
  │ DataExporter.exportAll()       │ 54   │ 14   │ 4    │ 134     │
  └────────────────────────────────┴───────┴──────┴──────┴─────────┘

📊  Complexity Breakdown
  ┌──────────────────────┬───────┬──────────┐
  │ Metric               │ Value │ Threshold│
  ├──────────────────────┼───────┼──────────┤
  │ Avg function length  │ 24    │ ≤ 20     │ 🟡
  │ Max nesting depth    │ 8     │ ≤ 4      │ 🔴
  │ Max cyclomatic comp  │ 34    │ ≤ 10     │ 🔴
  │ Functions > 100 lines│ 12    │ 0        │ 🔴
  │ Avg params per func  │ 2.4   │ ≤ 3      │ 🟢
  └──────────────────────┴───────┴──────────┘

⚠️  Chaos + Fossil = Tech Debt Epicenter
  PaymentGateway.process() — chaos score 87, fossil age 890d → refactor NOW
```

### `/code-chaos scan --thresholds <config>`
Custom thresholds: `--thresholds "branches=20,depth=5,lines=80"`

### `/code-chaos scan --detail <function>`
Deep dive into a specific function: show each branch, nesting tree, and suggestion.

## Implementation Notes

1. Parse function/method signatures per language:
   - JS/TS: `function`, `=>`, method syntax
   - Java: method signatures
   - Python: `def `
   - Go: `func`
   - Rust: `fn`
2. For each function measure:
   - **Cyclomatic complexity**: count `if`, `else`, `for`, `while`, `case`, `catch`, `&&`, `||`, `?`
   - **Nesting depth**: max indentation level inside the function body
   - **Line count**: total lines of the function
   - **Parameter count**: number of parameters
3. Composite score: weighted combination (branches × 2 + depth × 5 + lines × 0.3 + params × 3)
4. Present functions ranked by score descending

## Common Mistakes

- Arrow functions/callbacks inflate cyclomatic count: distinguish between function declaration and callback — only score declared functions
- Template literals with `${}` mis-parsed as expressions: skip string contents
- Ternary chains: `a ? b : c ? d : e` — count each `?` as a branch
- Switch statements: each `case` counts as one branch, but the whole switch is one decision point — note both metrics
