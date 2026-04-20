# Post-worktree hooks

Hooks live **in this git-work repository** (alongside the `work` script). After `work checkout` or `work create`, `work` runs scripts from `WORK_POST_WORKTREE_DIR` in `.env` (default: `.work/post-worktree.d` under the tooling checkout).

## Rules

- Put one script per task; they run in **lexical sort** order (use prefixes like `10-`, `20-`).
- A file runs if it is **executable** (`chmod +x`) or ends in **`.sh`** (invoked with `bash`).
- Other files (e.g. this README) are ignored.
- Hooks run with the **worktree root** as the current working directory.
- Exported variables: **`WORK_WORKTREE_PATH`**, **`WORK_HOOK_REASON`** (`checkout` or `create`).

## Example (bash)

Save as `10-example.sh`, then `chmod +x 10-example.sh`.

```bash
#!/bin/bash
set -euo pipefail
echo "post-worktree (${WORK_HOOK_REASON}): ${WORK_WORKTREE_PATH}"
```

Commit `.work/post-worktree.d/` in **git-work** so every machine using this checkout runs the same automation.
