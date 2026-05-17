# Code-Vitals 🩺

> 8 codebase analysis skills measuring every dimension of code health: age, frequency, churn, bus factor, scars, coupling, mutation, and complexity.

## Skills

| Skill | Tag | Dimension |
|-------|-----|-----------|
| **code-fossil** 🦴 | Age | Oldest surviving lines — what hasn't changed |
| **code-hotspot** 🔥 | Frequency | Most executed code — what runs the most |
| **code-churn** 📊 | Instability | Most frequently changed files — what churns |
| **code-soloist** 🎯 | Bus Factor | Single-author files — who owns what |
| **code-scar** 🩹 | Reverts | Most rolled-back files — history of mistakes |
| **code-sprawl** 🕸️ | Coupling | Fan-in/fan-out — dependencies and tight coupling |
| **code-mutation** 🧬 | Volume | Cumulative lines added/deleted — how much rewritten |
| **code-chaos** 🌪️ | Complexity | Cyclomatic complexity, nesting, line length |

## Compatibility

Works with any agent that supports the **Agent Skills Specification**:

| Agent | Supported |
|-------|-----------|
| Claude Code | ✅ |
| Codex CLI | ✅ |
| Cursor | ✅ |
| OpenCode | ✅ |
| GitHub Copilot | ✅ |
| Kiro CLI | ✅ |

## Installation

Clone the repo and copy skills to your agent's skills directory:

```bash
git clone https://github.com/icebears111/code-vitals.git
cd code-vitals
```

### OpenCode (global)

```bash
cp -r skills/* ~/.config/opencode/skills/
```

### Claude Code / Codex CLI / Cursor

```bash
cp -r skills/* ~/.claude/skills/
cp -r skills/* ~/.agents/skills/
```

### Project-local (any agent)

```bash
cp -r skills/* .opencode/skills/
```

### Single skill

```bash
cp -r skills/code-fossil ~/.config/opencode/skills/
cp -r skills/code-hotspot ~/.config/opencode/skills/
```

### Verify

Ask your agent:

> Scan this repo with code-fossil

## Cross-Reference Matrix

Combine skills to uncover deeper insights:

| Combo | Insight |
|-------|---------|
| fossil + hotspot | Old code that runs a lot → refactor priority |
| churn + sprawl | Files that change often AND are heavily depended on → biggest blast radius |
| soloist + churn | Single-author files that change fast → bus factor emergency |
| scar + chaos | Complex files with revert history → rewrite candidates |
| mutation + fossil | Files rewritten many times but still have old lines → architectural debt |
| hotspot + soloist | Hot code owned by one person → critical bus factor |

## License

MIT
