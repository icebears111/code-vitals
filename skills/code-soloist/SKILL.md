---
name: code-soloist
description: Use when assessing bus factor risk, finding files owned by a single developer, identifying knowledge silos, or planning cross-training. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Soloist 🎯

## What It Does
Finds files authored or last-touched by a single developer. These are bus-factor risks — if that person leaves, nobody knows that code.

## Commands

### `/code-soloist scan`
Scan all tracked files and count unique authors who touched each file.

```
═══════════════════════════════════════════
  CODE SOLOIST REPORT
═══════════════════════════════════════════

🎯  Single-Author Files (Bus Factor = 1)
  ┌──────────────────────────┬──────────┬────────┐
  │ File                     │ Author   │ Age    │
  ├──────────────────────────┼──────────┼────────┤
  │ src/payments/gateway.ts  │ alice    │ 423d   │ ← 🔴
  │ src/deploy/k8s-config.ts │ bob      │ 287d   │ ← 🔴
  │ src/auth/saml.ts         │ carol    │ 156d   │ ← 🔴
  │ legacy/migration/v1.sql  │ alice    │ 890d   │ ← 🔴
  │ docs/architecture.md     │ bob      │ 34d    │
  └──────────────────────────┴──────────┴────────┘

👥  Author Distribution
  ┌──────────┬──────────┬──────────┬────────┐
  │ Author   │ Files    │ Solo     │ Risk   │
  ├──────────┼──────────┼──────────┼────────┤
  │ alice    │ 47       │ 12       │ 🔴     │
  │ bob      │ 52       │ 8        │ 🟡     │
  │ carol    │ 38       │ 3        │ 🟢     │
  │ dave     │ 12       │ 0        │ 🟢     │
  └──────────┴──────────┴──────────┴────────┘

🔴  Critical: Solo File + High Churn
  src/payments/gateway.ts — only alice touches it, changes 3x/week
```

### `/code-soloist scan --risk`
Cross-reference with code-churn: solo files with above-average churn = critical risk.

### `/code-soloist scan --recent <days>`
Only consider changes in the last N days — filters out historical drive-bys.

## Implementation Notes

1. Use `git blame` or `git shortlog -ns -- <file>` per tracked source file
2. Count unique authors per file
3. Files with exactly 1 author = solo files
4. Cross-reference with churn data for --risk
5. Exclude generated files and lock files

## Common Mistakes

- Co-authored commits: `git log` handles co-authors via commit message trailers — parse `Co-authored-by:` if present
- Global committer vs actual author: use `--format=%an` (author name) not committer
- Drive-by commits inflate author count: a 1-line typo fix doesn't mean the author knows the file
