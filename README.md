# 37signals AI Skills

## Installation

```bash
# Pipe directly to an AI agent
curl -fsSL https://raw.githubusercontent.com/basecamp/ai/main/install.md | claude

# Or paste install.md into any coding assistant
```

> **Rolling main**: Installs from `main` branch, updates via `git pull`.

## Skills

General-purpose skills designed for broad use.

| Skill | Description |
|-------|-------------|
| [consult-outside-expert](skills/consult-outside-expert/) | Consult an outside expert (via Codex MCP) to collaboratively refine, stress-test, and converge on excellent outcomes |
| [install-md](skills/install-md/) | Create install.md files optimized for AI agent execution |
| [skill-crafting](skills/skill-crafting/) | Create and refine agent skills through co-development and eval loops |

## Updating

```bash
cd ~/.local/share/37signals-ai && git pull
```

Symlinks mean updates take effect immediately.

## License

MIT - see [MIT-LICENSE](MIT-LICENSE).
