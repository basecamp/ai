# 37signals AI Skills

Install selected skills. Execute autonomously.

> **Rolling main**: This repo is installed from `main` and updated via `git pull`.
> Skills update in place. Review commits if you need stability guarantees.

## Done when

- [ ] Repository present and up to date at `~/.local/share/37signals-ai`
- [ ] Skills directory resolved and created if needed
- [ ] Selected skill(s) symlinked to skills directory

## Steps

1. Clone or update the repository:
   ```bash
   if git -C ~/.local/share/37signals-ai rev-parse 2>/dev/null; then
     git -C ~/.local/share/37signals-ai pull
   else
     git clone https://github.com/basecamp/ai ~/.local/share/37signals-ai
   fi
   ```
   If the directory exists but isn't the expected repo, ask the user before overwriting.

2. Determine your skills directory:
   - Check your tool's configuration or documentation for the skills path
   - If unknown, ask the user
   - Fall back to `~/.claude/skills` (Claude Code default)

3. Set `SKILLS_DIR` to the resolved path:
   ```bash
   SKILLS_DIR="$HOME/.claude/skills"  # or resolved path
   ```

4. Create the skills directory and confirm:
   ```bash
   mkdir -p "$SKILLS_DIR"
   echo "Installing skills to: $SKILLS_DIR"
   ```

5. List available skills:
   ```bash
   ls ~/.local/share/37signals-ai/skills/
   ```

6. Ask which skill(s) to install

7. For each selected skill:
   ```bash
   ln -sfn ~/.local/share/37signals-ai/skills/SKILL_NAME "$SKILLS_DIR/SKILL_NAME"
   ```

## Updating

```bash
cd ~/.local/share/37signals-ai && git pull
```

Symlinks mean updates take effect immediately.

---

[install.md spec](https://www.installmd.org)
