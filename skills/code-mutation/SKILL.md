---
name: code-mutation
description: Use when measuring cumulative code change volume, finding files with the most lines added/deleted over time, analyzing rewrite patterns, or identifying high-velocity code. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Mutation 🧬

## What It Does
Measures cumulative change volume per file — total lines added + deleted over the entire git history. High mutation = most rewritten, highest churn in terms of code mass.

## Commands

### `/code-mutation scan`
Count total lines added and deleted per file across all commits.

```
═══════════════════════════════════════════
  CODE MUTATION REPORT
═══════════════════════════════════════════

🧬  Highest Mutation Files
  ┌──────────────────────────┬────────┬─────────┬─────────┐
  │ File                     │ Added  │ Deleted │ Total   │
  ├──────────────────────────┼────────┼─────────┼─────────┤
  │ src/core/engine.ts       │ 12,430 │ 8,912   │ 21,342  │ ← 🔴
  │ src/api/router.ts        │ 8,211  │ 5,432   │ 13,643  │
  │ src/components/UI.tsx    │ 6,543  │ 3,210   │ 9,753   │
  │ legacy/migration_v2.sql  │ 5,001  │ 4,892   │ 9,893   │
  │ src/utils/helpers.ts     │ 3,924  │ 2,876   │ 6,800   │
  └──────────────────────────┴────────┴─────────┴─────────┘

📈  Mutation vs Current Size
  ┌──────────────────────┬──────────┬──────────┬──────┐
  │ File                 │ Current  │ Lifetime │ Churn│
  │                      │ Lines    │ Written  │ Ratio│
  ├──────────────────────┼──────────┼──────────┼──────┤
  │ src/core/engine.ts   │ 847      │ 21,342   │ 25.2 │ ← rewritten 25x
  │ src/api/router.ts    │ 534      │ 13,643   │ 25.5 │
  │ src/utils/helpers.ts │ 312      │ 6,800    │ 21.8 │
  └──────────────────────┴──────────┴──────────┴──────┘

⚠️  High Mutation + Low Test Coverage = Rewrite candidate
```

### `/code-mutation scan --ratio`
Show churn ratio (lifetime written / current size). Files with ratio > 10 have been completely rewritten multiple times.

### `/code-mutation scan --since <date>`
Limit to recent N days. Useful for measuring pace of change.

## Implementation Notes

1. Use `git log --numstat --pretty=format: --no-merges` to get per-file +/− counts
2. Parse added/deleted columns per file path
3. Sum across all commits, sort by total
4. For `--ratio`: current size = `wc -l <file>`, then total / current = churn ratio
5. Exclude binary files and lock files (they inflate mutation)

## Common Mistakes

- Binary files (images, PDFs) counted as single-line changes in numstat — skip them by checking `git diff --numstat` returns `-` instead of numbers
- File renames reset history: `--follow` helps but is slow — note in report if a file appears young but has high mutation (likely renamed)
- Merge commits create noise: always use `--no-merges`
