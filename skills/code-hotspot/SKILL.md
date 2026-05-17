---
name: code-hotspot
description: Use when identifying which code runs most frequently, analyzing execution hotspots, combining coverage data with code age to find high-risk areas. Pair with code-fossil for cold-storage twin view. Compatible with Claude Code, Codex, Cursor, OpenCode, GitHub Copilot, Kiro CLI.
---

# Code-Hotspot 🔥

## What It Does
Finds the hottest code in your codebase — the lines/functions that execute most frequently. Pair with `code-fossil` for the complete picture: fossil shows what hasn't changed, hotspot shows what runs the most.

## When to Use
- You want to know which code paths execute most often
- Performance optimization: find hot paths to optimize first
- Risk analysis: combine with code-fossil — "this code is ancient AND runs 10k times/sec" = refactor priority
- Test priority: find untested hotspots (high execution, low coverage)
- Onboarding: "these 5 files handle 80% of all requests"

## How It Works

The agent reads whatever execution data is available in the project, in order of preference:

1. **Coverage reports** — JaCoCo HTML/XML, pytest `.coverage`, Jest `lcov.info`, Go `cover.out`
2. **Profiler output** — async-profiler HTML, pprof, Valgrind callgrind
3. **APM data** — OpenTelemetry traces (if available)
4. **Manual fallback** — guide the user to generate data if nothing exists

## Commands

### `/code-hotspot scan`
Auto-detect available execution data and generate a hotspot report.

```
═══════════════════════════════════════════
  CODE HOTSPOT REPORT
═══════════════════════════════════════════

🔥  Hottest Functions (execution count)
  1. UserService.findById()    — 142,503 calls
  2. OrderController.create()  — 98,214 calls
  3. AuthMiddleware.verify()   — 87,441 calls
  4. PaymentGateway.charge()   — 12,302 calls
  5. ReportGenerator.export()  — 342 calls

⚠️  High Risk (hot + old + no test)
  ┌──────────────────────┬────────┬────────┬───────┐
  │ Function             │ Calls  │ Age    │ Tests │
  ├──────────────────────┼────────┼────────┼───────┤
  │ PaymentGateway.charge│ 12k/s  │ 890d   │ ❌   │ ← 🔥
  │ AuthMiddleware.verify│ 87k/s  │ 654d   │ ✅   │
  │ CacheManager.evict() │ 45k/s  │ 120d   │ ❌   │ ← 🔥
  └──────────────────────┴────────┴────────┴───────┘
```

### `/code-hotspot scan --risk`
Cross-reference with `git blame` age. Identifies code that is both old and hot — highest refactoring ROI.

### `/code-hotspot scan --cold`
Invert: find code with zero or near-zero execution — candidates for deletion or dead code cleanup. Pairs with `code-fossil graveyard`.

### `/code-hotspot guide`
If no execution data is found, show the user exactly how to generate it for their stack.

## Supported Data Sources

| Tool | Language | Auto-detected |
|------|----------|:---:|
| JaCoCo (`target/site/jacoco/`) | Java | ✅ |
| pytest (`--coverage`) | Python | ✅ |
| Jest / Vitest (`lcov.info`) | JS/TS | ✅ |
| Go `cover.out` | Go | ✅ |
| async-profiler | Java | ⚠️ file path pattern |
| pprof | Go | ⚠️ file path pattern |
| cargo-tarpaulin | Rust | ✅ |
| dotCover | C# | ⚠️ |
| OpenTelemetry traces | Any | ⚠️ via span data |

## Implementation Notes for the Agent

1. First, search for coverage/profiling data files in the project:
   - `**/jacoco/**/*.xml` or `**/site/jacoco/**`
   - `**/coverage/lcov.info`
   - `**/.coverage` (Python)
   - `**/cover.out` (Go)
   - `**/cobertura.xml`
   - `**/profile/*.html` or `**/flamegraph/*.html`
2. Parse the data into a structured (file, line, hit-count) form
3. For `--risk`, cross-reference with `git blame` age (import from code-fossil)
4. For `--cold`, find functions with 0-1 hits in coverage data
5. If no data exists, fall back to `guide` — don't guess or fabricate numbers
6. Always report the data source and its timestamp so the user knows how fresh it is

## Common Mistakes

- Confusing coverage with profiling: coverage tells you what ran at least once in the test suite, profiling tells you actual call frequency — treat them differently
- Stale reports: check file modification time, warn if data is older than 7 days
- Generating fake data: if no execution data exists, DO NOT approximate — tell the user how to generate it
- Binary/unreadable formats: profiler raw output (.jfr, .bin) needs conversion first — guide the user

## Limitations

- Requires the project to have execution data (coverage run or profiler) — it's not magic, it reads reports
- Different tools report in different units (hits vs. samples vs. percent) — comparisons across tools are approximate
- Only as accurate as the data it's given — stale data = stale conclusions
