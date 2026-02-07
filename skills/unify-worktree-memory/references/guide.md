# Worktree Memory Consolidation Guide

Consolidate Claude Code memory across git worktrees so all branches share a single memory directory.

---

## The Problem

Claude Code stores per-project memory at `~/.claude/projects/<encoded-path>/memory/`. The path is derived from the working directory — each git worktree gets a unique encoded path, so every worktree gets isolated memory. You lose all accumulated knowledge when switching branches.

**There is no built-in config to customize the memory path.** The solution is symlinks: replace each worktree's `memory/` dir with a symlink to the base repo's `memory/` dir. Claude Code follows symlinks transparently.

## Path Encoding

Claude Code encodes the working directory path by replacing `/` and `.` with `-`:

| Working directory | Encoded project dir |
|---|---|
| `/Users/you/Work/my-app` | `-Users-you-Work-my-app` |
| `/Users/you/.worktrees/my-app/feature` | `-Users-you--worktrees-my-app-feature` |

Note: a path containing `/.` produces `--` (one dash from `/`, one from `.`).

## Architecture

Two scripts, one hook:

1. **`@scripts/consolidate-memory`** — One-time migration for existing worktrees
2. **`@scripts/ensure-memory-symlink`** — SessionStart hook for future worktrees
3. **`~/.claude/settings.json`** hook entry — Triggers the hook on every session start

### How It Works

For a worktree at `/Users/you/.worktrees/my-app/feature-branch`:

1. `git rev-parse --git-common-dir` resolves to the base repo's `.git` dir
2. Strip `/.git` to get the base repo path (e.g., `/Users/you/Work/my-app`)
3. Encode both paths to get their Claude project dir names
4. Ensure the base repo's `memory/` dir exists
5. If the worktree has existing memory content, merge it into the base
6. Replace the worktree's `memory/` dir with a symlink to the base's `memory/`

---

## Implementation

### Step 1: Customize the Consolidation Script

Open `@scripts/consolidate-memory`. Change `WORKTREE_ROOT` to point to your worktree root directory:

```bash
WORKTREE_ROOT="$HOME/.worktrees"  # <-- customize this
```

**Design decisions in this script:**

**Two-phase processing per repo:**
- Phase 1: Process worktrees that still exist on disk. Uses the `.git` file in each worktree to authoritatively resolve the base repo. This is always correct.
- Phase 2: Process orphaned Claude project dirs (worktrees deleted from disk). Uses encoded path prefix matching with disambiguation.

**Prefix collision disambiguation:**
Repo names like `my-app` and `my-app-api` create ambiguous encoded prefixes. The worktree dir `my-app/api-feature` encodes identically to `my-app-api/feature`. Phase 2 handles this by only deferring to repos with *strictly longer* names — if `my-app-api` also exists as a repo, its prefix takes priority for matching orphaned dirs.

**Content merging:**
When a worktree has MEMORY.md content and the base repo also has content, the worktree content is appended with a `## From worktree: <branch>` header. Topic files with name conflicts are renamed with a branch suffix to avoid data loss.

**Dry run support:**
Pass `--dry-run` to preview all actions without making changes.

### Step 2: Register the SessionStart Hook

The hook script is `@scripts/ensure-memory-symlink`. It runs on every Claude Code session start. The fast path (non-worktree sessions) completes in ~5ms.

Add to `~/.claude/settings.json`, using the absolute path to the script:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "<absolute-path-to-skill>/scripts/ensure-memory-symlink",
            "timeout": 5000
          }
        ]
      }
    ]
  }
}
```

Resolve `<absolute-path-to-skill>` — if installed via `npx skills add`, the skill is symlinked at `~/.claude/skills/unify-worktree-memory/`, so use:

```
~/.claude/skills/unify-worktree-memory/scripts/ensure-memory-symlink
```

### Step 3: Run the Migration

```bash
# Make scripts executable (if not already)
chmod +x <absolute-path-to-skill>/scripts/consolidate-memory
chmod +x <absolute-path-to-skill>/scripts/ensure-memory-symlink

# Preview what will happen
<absolute-path-to-skill>/scripts/consolidate-memory --dry-run

# Run for real
<absolute-path-to-skill>/scripts/consolidate-memory
```

The script is idempotent — running it again skips already-linked dirs.

---

## Customization

**`WORKTREE_ROOT`** in the consolidation script must point to your worktree root directory. The hook script doesn't need this — it uses `git rev-parse` to detect worktrees dynamically.

**Multiple worktree roots:** If you use worktrees in different locations, the hook handles this automatically (it's per-session, based on `cwd`). The consolidation script would need to be run once per root, or modified to iterate over multiple roots.

---

## Verification

After running the consolidation:

```bash
# Check symlinks point to base repo memory
ls -la ~/.claude/projects/*worktree*/memory

# Write to memory in one worktree, verify in another
# (start Claude Code in worktree A, write to memory, then check in worktree B)

# Verify the hook works for new worktrees
git worktree add /path/to/new-worktree feature-branch
# Start Claude Code in /path/to/new-worktree
# The hook auto-creates the symlink
```

---

## Edge Cases

| Scenario | Behavior |
|---|---|
| Base project dir doesn't exist | Created with `memory/` subdir |
| Worktree has content, base doesn't | Content moved to base before symlinking |
| Both have content | Worktree content appended to base with `## From worktree:` header |
| Non-git directory | Hook exits immediately (fast path) |
| Regular repo (not a worktree) | Hook exits immediately (fast path) |
| Already symlinked | Skipped (idempotent) |
| Orphaned worktree (deleted from disk) | Consolidation script handles via prefix matching |
| Ambiguous repo name prefix (`app` vs `app-api`) | Only repos with strictly longer names can claim orphaned dirs |
