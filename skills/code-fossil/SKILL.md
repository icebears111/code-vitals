---
name: code-fossil
description: Use when exploring codebase history, identifying ancient surviving code, visualizing code age distribution, or tracing the oldest unchanged lines across a repository. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI, and any agent supporting the Agent Skills Specification.
---

# Code-Fossil

## What It Does
Archaeology for codebases. Finds the oldest surviving lines of code still present in your repository, traces their lineage, and generates visual reports showing age distribution across files and modules.

## When to Use
- You want to know which line of code has survived the longest without change
- You need a "fossil map" showing which parts of the codebase are ancient vs recently rewritten
- Onboarding: show new team members the historical layers of the project
- Legacy audit: identify which files have accumulated the most unchanged code
- Curiosity: find the birth date of every file and the single oldest live line

## How It Works

The agent runs git commands (no external dependencies) and produces reports:

1. **Scan** `git blame` across all tracked source files
2. **Filter** skip package declarations, import statements, blank lines, and standalone comments — only score lines that contain actual logic
3. **Rank** lines by commit date ascending
4. **Trace** the oldest surviving line — find its commit, author, date, and message
5. **Aggregate** per-file statistics (oldest line per file, average age, distribution)
6. **Present** results as a structured report

## Commands

### `/code-fossil scan`
Full repository scan. Produces:

```
═══════════════════════════════════════════
  CODE FOSSIL REPORT
═══════════════════════════════════════════

🏆  Oldest Surviving Line
  File:   src/core/engine.ts:47
  Age:    1263 days (3y 5m 18d)
  Author: alice@example.com
  Date:   2023-01-15
  Commit: a3f2b1c "Initial engine implementation"
  Code:   export class Engine {

📊  Fossil Layers (by year)
  ┌──────────┬──────────┬────────┐
  │ Layer    │ Files    │ Lines  │
  ├──────────┼──────────┼────────┤
  │ 2024+    │ 142      │ 8,341  │ Recent
  │ 2023     │ 89       │ 4,217  │ │
  │ 2022     │ 34       │ 1,056  │ ▼ older
  │ 2021     │ 12       │ 312    │
  │ 2020     │ 3        │ 47     │ Ancient
  └──────────┴──────────┴────────┘

🦴  Top 5 Fossils (oldest files)
  1. src/utils/helpers.ts    — 1,480 days (alice)
  2. src/core/engine.ts      — 1,263 days (bob)
  3. src/config/defaults.ts  — 1,102 days (alice)
  4. src/types/index.ts      — 998 days  (carol)
  5. src/api/router.ts       — 876 days  (bob)
```

### `/code-fossil scan --deep`
Full scan that includes line-by-line age analysis per file. Outputs JSON dataset for further processing.

### `/code-fossil scan --html`
Generates a self-contained HTML fossil map with color-coded layers for browser viewing.

### `/code-fossil trace <filepath>`
Trace a single file's fossil history. Shows which lines are from which era.

Example output:

```
📜  src/components/Button.tsx
  ─────────────────────────────────────────
  Lines 1-12   (2025)   import/react setup
  Lines 13-47  (2023)   core button logic ← fossil layer
  Lines 48-52  (2026)   new variant support
  Lines 53-55  (2022)   style constants    ← oldest in file (1,185 days)
  ─────────────────────────────────────────
  Oldest line:  line 53  "const colors = {"
  Youngest:     line 48  (2 days ago)
  Average age:  413 days
```

### `/code-fossil graveyard`
Show deleted files and the commits that killed them (git log --diff-filter=D).

## Implementation Notes for the Agent

When the user invokes this skill:

1. Always start from the repository root
2. Use `git blame` with date format: `git blame --date=short <file>`
3. Parse the blame output to extract commit dates per line
4. **Filter out non-logic lines**: skip `package`, `import`, blank lines, `//` comments, `/* */` block comments, `@` annotations, `)` closing braces on their own line — only count lines that contain actual code logic
5. Use `git log --follow --format="%H %ai %an %s" -1 <file>` to trace renames
6. For large repos, focus on tracked source files (exclude node_modules, dist, vendor)
7. Present results in the structured format above
8. For `--html`, generate a standalone HTML file with inline CSS

## Common Mistakes

- Blaming binary/generated files: skip `*.min.js`, `*.bundle.*`, lock files, and build artifacts
- Blaming 10,000+ files: use `--since` flag to limit scope, or focus on src/ directory
- Renamed files: `git blame --follow` handles renames, but is slower — use it only for `trace`
- Empty blame: file may be new or untracked — check `git log --oneline <file>` first
- **Package/import lines reported as fossils**: always filter out `package`, `import`, blank lines, comments, and annotations before ranking — only actual logic lines count

## Limitations

- Only works in git repositories
- Large monorepos may take 30+ seconds for a full scan
- Squashed commits reset "age" — oldest surviving line dates from the squash, not original authoring
