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

1. **`~/.claude/bin/consolidate-memory`** — One-time migration for existing worktrees
2. **`~/.claude/bin/ensure-memory-symlink`** — SessionStart hook for future worktrees
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

### Step 1: Create the Consolidation Script

Create `~/.claude/bin/consolidate-memory` and make it executable.

This script iterates over a worktree root directory (customize `WORKTREE_ROOT` for your setup), finds all git worktrees, resolves their base repos, and creates symlinks.

Key design decisions:

**Two-phase processing per repo:**
- Phase 1: Process worktrees that still exist on disk. Uses the `.git` file in each worktree to authoritatively resolve the base repo. This is always correct.
- Phase 2: Process orphaned Claude project dirs (worktrees deleted from disk). Uses encoded path prefix matching with disambiguation.

**Prefix collision disambiguation:**
Repo names like `my-app` and `my-app-api` create ambiguous encoded prefixes. The worktree dir `my-app/api-feature` encodes identically to `my-app-api/feature`. Phase 2 handles this by only deferring to repos with *strictly longer* names — if `my-app-api` also exists as a repo, its prefix takes priority for matching orphaned dirs.

**Content merging:**
When a worktree has MEMORY.md content and the base repo also has content, the worktree content is appended with a `## From worktree: <branch>` header. Topic files (anything other than MEMORY.md) are copied only if they don't already exist in the base.

**Dry run support:**
Pass `--dry-run` to preview all actions without making changes.

```bash
#!/usr/bin/env bash
set -euo pipefail

CLAUDE_PROJECTS="$HOME/.claude/projects"
WORKTREE_ROOT="$HOME/.worktrees"  # <-- customize this
DRY_RUN=false

if [[ "${1:-}" == "--dry-run" ]]; then
  DRY_RUN=true
  echo "=== DRY RUN ==="
  echo
fi

encode_path() {
  echo "$1" | sed 's|/|-|g; s|\.|-|g'
}

# Collect all repo names for prefix disambiguation
repo_names=()
for repo_dir in "$WORKTREE_ROOT"/*/; do
  repo_names+=("$(basename "$repo_dir")")
done

merged=0
symlinked=0
skipped=0

process_worktree_memory() {
  local wt_project_dir="$1"
  local wt_memory="$wt_project_dir/memory"
  local wt_name="$2"
  local branch_hint="$3"
  local base_memory="$4"

  if [[ -L "$wt_memory" ]]; then
    echo "  ALREADY LINKED: $wt_name"
    ((skipped++)) || true
    return
  fi

  if [[ ! -d "$wt_memory" ]]; then
    if [[ "$DRY_RUN" == false ]]; then
      ln -s "$base_memory" "$wt_memory"
    fi
    echo "  LINKED (empty): $wt_name"
    ((symlinked++)) || true
    return
  fi

  # Merge MEMORY.md
  if [[ -f "$wt_memory/MEMORY.md" ]]; then
    if [[ "$DRY_RUN" == false ]]; then
      {
        echo ""
        echo "## From worktree: $branch_hint"
        echo ""
        cat "$wt_memory/MEMORY.md"
      } >> "$base_memory/MEMORY.md"
    fi
    echo "  MERGED MEMORY.md: $wt_name ($branch_hint)"
    ((merged++)) || true
  fi

  # Copy topic files (rename on conflict to avoid data loss)
  while IFS= read -r topic_file; do
    fname=$(basename "$topic_file")
    [[ "$fname" == "MEMORY.md" ]] && continue
    if [[ -f "$base_memory/$fname" ]]; then
      # Rename with branch hint to preserve both versions
      dest="${fname%.*}-${branch_hint}.${fname##*.}"
      if [[ "$DRY_RUN" == false ]]; then
        cp "$topic_file" "$base_memory/$dest"
      fi
      echo "  COPIED topic (renamed): $fname -> $dest"
    else
      if [[ "$DRY_RUN" == false ]]; then
        cp "$topic_file" "$base_memory/$fname"
      fi
      echo "  COPIED topic: $fname"
    fi
  done < <(find "$wt_memory" -maxdepth 1 -type f 2>/dev/null)

  if [[ "$DRY_RUN" == false ]]; then
    rm -rf "$wt_memory"
    ln -s "$base_memory" "$wt_memory"
  fi
  echo "  LINKED: $wt_name"
  ((symlinked++)) || true
}

for repo_dir in "$WORKTREE_ROOT"/*/; do
  repo_name=$(basename "$repo_dir")

  # Resolve base repo from any worktree's .git file
  base_repo=""
  while IFS= read -r git_file; do
    gitdir_line=$(cat "$git_file")
    base_repo=$(echo "$gitdir_line" | sed 's|^gitdir: ||; s|/\.git/worktrees/.*||')
    break
  done < <(find "$repo_dir" -name ".git" -type f -maxdepth 4 2>/dev/null)

  if [[ -z "$base_repo" ]]; then
    echo "SKIP $repo_name: no worktrees with .git file found"
    ((skipped++)) || true
    continue
  fi

  base_project_dir="$CLAUDE_PROJECTS/$(encode_path "$base_repo")"
  base_memory="$base_project_dir/memory"

  echo "--- $repo_name ---"
  echo "  base repo: $base_repo"

  if [[ "$DRY_RUN" == false ]]; then
    mkdir -p "$base_memory"
  fi

  # Phase 1: Existing worktrees (authoritative via .git file)
  while IFS= read -r git_file; do
    wt_path=$(dirname "$git_file")
    wt_encoded="$(encode_path "$wt_path")"
    wt_project_dir="$CLAUDE_PROJECTS/$wt_encoded"
    [[ -d "$wt_project_dir" ]] || continue
    process_worktree_memory "$wt_project_dir" "$wt_encoded" "$(basename "$wt_path")" "$base_memory"
  done < <(find "$repo_dir" -name ".git" -type f -maxdepth 4 2>/dev/null)

  # Phase 2: Orphaned project dirs (prefix matching with disambiguation)
  encoded_wt_root="$(encode_path "$WORKTREE_ROOT/$repo_name")"
  for wt_project_dir in "$CLAUDE_PROJECTS/$encoded_wt_root"-*/; do
    [[ -d "$wt_project_dir" ]] || continue
    wt_name=$(basename "$wt_project_dir")

    [[ -L "$wt_project_dir/memory" ]] && continue

    # Only defer to repos with strictly longer names
    claimed_by_longer=false
    for other_repo in "${repo_names[@]}"; do
      [[ "$other_repo" == "$repo_name" ]] && continue
      [[ ${#other_repo} -le ${#repo_name} ]] && continue
      other_prefix="$(encode_path "$WORKTREE_ROOT/$other_repo")"
      if [[ "$wt_name" == "$other_prefix"-* ]]; then
        claimed_by_longer=true
        break
      fi
    done
    [[ "$claimed_by_longer" == true ]] && continue

    branch_hint=$(echo "$wt_name" | sed "s|^${encoded_wt_root}-||")
    process_worktree_memory "$wt_project_dir" "$wt_name" "$branch_hint" "$base_memory"
  done

  echo
done

echo "=== Summary ==="
echo "  Merged: $merged | Symlinked: $symlinked | Skipped: $skipped"
[[ "$DRY_RUN" == true ]] && echo -e "\nRe-run without --dry-run to apply."
```

### Step 2: Create the SessionStart Hook Script

Create `~/.claude/bin/ensure-memory-symlink` and make it executable.

This runs on every Claude Code session start. The fast path (non-worktree sessions) completes in ~16ms.

```bash
#!/usr/bin/env bash
set -euo pipefail

CLAUDE_PROJECTS="$HOME/.claude/projects"

input=$(cat)
cwd=$(echo "$input" | jq -r '.cwd // empty')
[[ -z "$cwd" ]] && exit 0

# Fast path: check if this is a git worktree
git_common_dir=$(git -C "$cwd" rev-parse --git-common-dir 2>/dev/null) || exit 0
git_dir=$(git -C "$cwd" rev-parse --git-dir 2>/dev/null) || exit 0

git_common_dir=$(cd "$cwd" && cd "$git_common_dir" && pwd)
git_dir_abs=$(cd "$cwd" && cd "$git_dir" && pwd)

# Not a worktree — git-dir and git-common-dir are the same
[[ "$git_common_dir" == "$git_dir_abs" ]] && exit 0

# Resolve base repo path
base_repo=$(echo "$git_common_dir" | sed 's|/\.git$||')

encode_path() {
  echo "$1" | sed 's|/|-|g; s|\.|-|g'
}

base_project_dir="$CLAUDE_PROJECTS/$(encode_path "$base_repo")"
base_memory="$base_project_dir/memory"
cwd_project_dir="$CLAUDE_PROJECTS/$(encode_path "$cwd")"
cwd_memory="$cwd_project_dir/memory"

# Already a symlink — done
[[ -L "$cwd_memory" ]] && exit 0

mkdir -p "$base_memory"

# Move existing worktree memory content to base before symlinking
if [[ -d "$cwd_memory" ]]; then
  if [[ -f "$cwd_memory/MEMORY.md" ]]; then
    if [[ -f "$base_memory/MEMORY.md" ]]; then
      {
        printf '\n## From worktree: %s\n\n' "$(basename "$cwd")"
        cat "$cwd_memory/MEMORY.md"
      } >> "$base_memory/MEMORY.md"
    else
      mv "$cwd_memory/MEMORY.md" "$base_memory/MEMORY.md"
    fi
  fi

  for f in "$cwd_memory"/*; do
    [[ -e "$f" ]] || continue
    fname=$(basename "$f")
    [[ "$fname" == "MEMORY.md" ]] && continue
    if [[ -e "$base_memory/$fname" ]]; then
      # Rename with worktree dir name to preserve both versions
      wt_hint=$(basename "$cwd")
      mv "$f" "$base_memory/${fname%.*}-${wt_hint}.${fname##*.}"
    else
      mv "$f" "$base_memory/$fname"
    fi
  done

  rm -rf "$cwd_memory"
fi

mkdir -p "$cwd_project_dir"
ln -s "$base_memory" "$cwd_memory"
```

### Step 3: Register the SessionStart Hook

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "/Users/you/.claude/bin/ensure-memory-symlink",
            "timeout": 5000
          }
        ]
      }
    ]
  }
}
```

Adjust the path to match your home directory.

### Step 4: Run the Migration

```bash
chmod +x ~/.claude/bin/consolidate-memory ~/.claude/bin/ensure-memory-symlink

# Preview what will happen
~/.claude/bin/consolidate-memory --dry-run

# Run for real
~/.claude/bin/consolidate-memory
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
