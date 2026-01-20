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

_None yet._

## Internal

Skills designed for 37signals internal use. Open source but not intended for external adoption—they reference internal paths, conventions, and context.

| Skill | Description |
|-------|-------------|
| [llms-txt](skills/llms-txt/) | Generate agent-accessible llms.txt tailored for 37signals sites |

## Updating

```bash
cd ~/.local/share/37signals-ai && git pull
```

Symlinks mean updates take effect immediately.

## License

MIT - see [MIT-LICENSE](MIT-LICENSE).
