---
name: code-churn
description: Use when identifying which files change most frequently, analyzing code instability, finding high-churn hotspots, or prioritizing refactoring targets by change frequency. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Churn 📊

## What It Does
Measures file modification frequency across git history. High churn = high maintenance cost. Find out which files change the most, how often, and in which patterns.

## Commands

### `/code-churn scan`
Count commits per file across the entire history.

```
═══════════════════════════════════════════
  CODE CHURN REPORT
═══════════════════════════════════════════

📊  Top 10 Highest Churn Files
  ┌──────────────────────────┬───────┬───────────┐
  │ File                     │ Edits │ Frequency │
  ├──────────────────────────┼───────┼───────────┤
  │ src/api/router.ts        │ 142   │ 2.1/week  │
  │ src/core/engine.ts       │ 98    │ 1.4/week  │
  │ config/deployment.yml    │ 87    │ 1.3/week  │
  │ src/utils/helpers.ts     │ 76    │ 1.1/week  │
  │ tests/e2e/flow.test.ts   │ 65    │ 0.9/week  │
  └──────────────────────────┴───────┴───────────┘

📈  Churn by Module
  ├── src/api/         — 312 edits (32%)
  ├── src/core/        — 245 edits (25%)
  ├── config/          — 189 edits (19%)
  ├── tests/           — 134 edits (14%)
  └── docs/            — 98 edits  (10%)

⚠️  High Risk: High Churn + Old Code
  cross-reference with code-fossil — files that change a lot AND are old
```

### `/code-churn scan --period <days>`
Limit to recent N days. Useful for sprint-level analysis.

### `/code-churn scan --author <name>`
Show which files a specific author changes most.

### `/code-churn scan --hotspot`
Cross-reference with coverage data: high churn + low test coverage = danger zone.

## Implementation Notes

1. Use `git log --name-only --pretty=format: --since=<date>` to get file list per commit
2. Count occurrences per file, sort descending
3. Calculate weekly frequency: total edits / (total span in days / 7)
4. For --period: add `--since=<date>` or `--after=<date>`
5. For --author: add `--author=<name>`
6. For --hotspot: if coverage data exists (see code-hotspot), cross-reference low-coverage files

## Common Mistakes

- Binaries/generated files inflate churn: exclude lock files, dist, minified assets
- Renamed files count as two: use `git log --follow` for accurate per-file tracking
- Merge commits double-count changes: filter with `--no-merges`
