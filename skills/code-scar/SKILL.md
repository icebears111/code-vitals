---
name: code-scar
description: Use when analyzing revert history, identifying error-prone files that get rolled back, discovering unstable code patterns, or auditing hotfix frequency. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Scar 🩹

## What It Does
Tracks revert history — files that have been rolled back most often. Each revert is a "scar" marking a mistake that made it past review. High scar count = process failure or inherently risky code.

## Commands

### `/code-scar scan`
Find all revert commits and the files they touched.

```
═══════════════════════════════════════════
  CODE SCAR REPORT
═══════════════════════════════════════════

🩹  Most Reverted Files
  ┌──────────────────────────┬───────┬──────────┐
  │ File                     │ Scars │ Last     │
  ├──────────────────────────┼───────┼──────────┤
  │ src/deploy/release.ts    │ 7     │ 2026-04  │ ← 🔴
  │ config/feature-flags.yml │ 5     │ 2026-05  │ ← 🔴
  │ src/api/migration.ts     │ 4     │ 2026-03  │
  │ src/db/schema.sql        │ 3     │ 2026-01  │
  │ ...                      │       │          │
  └──────────────────────────┴───────┴──────────┘

🔄  Revert Timeline
  2026-05-12  config/feature-flags.yml  "Revert: flag broke prod"
  2026-04-28  src/deploy/release.ts     "Revert: deployment failed"
  2026-04-15  src/deploy/release.ts     "Revert: rollback v3.2.1"
  2026-03-22  src/api/migration.ts      "Revert: data loss risk"

⚠️  Scar Pattern: Same file, same author
  src/deploy/release.ts — 5/7 reverts by bob → needs review process change
```

### `/code-scar scan --author`
Group reverts by author. Find who's causing the most rollbacks.

### `/code-scar scan --reason <keyword>`
Filter reverts by reason keyword (e.g. "perf", "security", "data loss").

### `/code-scar scan --since <date>`
Only consider reverts after a specific date.

## Implementation Notes

1. Find revert commits: `git log --grep="revert" --pretty=format:"%H %ai %an %s" --all`
2. For each revert commit, get changed files: `git diff-tree --no-commit-id --name-only -r <commit>`
3. Count occurrences per file
4. Also parse "Revert" commits that use the standard `git revert` message format
5. Optionally check if the original commit and revert share the same author

## Common Mistakes

- Case sensitivity: search for "Revert", "revert", "REVERT", "rollback", "backout"
- Merge reverts: `git revert -m` creates merge reverts — still count them
- Manual reverts: not all reverts use the word "revert" — also search for "rollback", "undo", "back out"
