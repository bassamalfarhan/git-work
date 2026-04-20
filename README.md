# git-work

Bash shortcuts around **`git worktree`** and related flows (`work` script).

## Background and workflow

Working on several issues at once usually means **multiple working trees**: one long-lived clone plus extra checkouts so each branch has its own directory. The underlying Git commands are powerful but awkward day to day: long **`git worktree add`** lines, remembering **two different paths** (the “main” repo vs each worktree), **fetching** and **grepping** `remotes/origin/...` to find the right branch, and **switching shells** into the right folder after each add.

**git-work** is a small Bash layer on top of that: you configure **one main clone** (`MAIN_PATH`) and **one parent directory** for all issue worktrees (`BRANCHES_PATH`). With **`JIRA_BASE`** set, the same branch names tie together **opening the ticket** (`work details`), **discovering what exists on `origin`** (`work checkout` after a fetch), **landing in the right tree** (`work` + grep / `work main`), and **shipping changes** (`work push`).

### Jira, branches, checkout, and push (how `work` ties it together)

Remote branches are expected to carry your **issue id** in the name (for example `ABC123-fix-login`). **`work checkout`** uses that naming: it **fetches**, shows matching **`remotes/origin/...`** lines, then adds a **worktree** under `BRANCHES_PATH` after you confirm. From a tree (or after jumping with **`work 123`**), **`work details`** builds **`JIRA_BASE` + id** and opens the browser; **`work push`** pushes the **current branch** from whatever directory you are in (still using `MAIN_PATH` as the git hub). **`work`** with no args lists what is already checked out locally.

### Typical flow (illustrative)

1. **One-time**: configure `.env` (including **`JIRA_BASE`** if you use Jira links), run **`work init`** (directories, clone if needed, install the **`work()`** shell wrapper).
2. **Often**: **`work`** lists what is already checked out; **`work 123`** jumps to the matching issue directory; **`work main`** (or your **`WORK_MAIN_ALIAS`**) returns to the main clone; **`work details`** (optionally with the same grep) opens the ticket in the browser from the branch name.
3. **New branch on disk**: **`work checkout 456`** fetches from `origin`, shows matching remote branches, then runs **`git worktree add`** under `BRANCHES_PATH` after you confirm (and can **`cd`** there when **`AUTO_CD_AFTER_CHECKOUT=true`**).
4. **Ship**: from inside a worktree, **`work push`** pushes the current branch to **`origin`** (setting upstream when needed).

The blocks below are **fabricated** examples of the shape of real output; your paths, prefixes, and counts will differ.

Bare **`work`** (list checked-out worktrees under `BRANCHES_PATH`):

```text
Checked out branches (3):

	[M] modified working tree  [A] ahead of upstream  [B] both  [C] clean

	ISSUES:
	ABC123-login-spinner [M]
	ABC127-api-timeout [C]

	RELEASES:
	release-2.4 [C]

	OTHER:
	spike-json-parser [A]

	For the base branch type main
```

**`work checkout`** (fetch, list remotes grouped like the bare list, then confirm before **`git worktree add`** — only proceeds when exactly one remote branch matches):

```text
Fetching from origin...


Listing available / matching branches

ISSUES:

	ABC123-login-spinner

RELEASES:

OTHER:


Will run the following in (/home/you/projects/acme-app):

	git worktree add /home/you/trees/acme/ABC123-login-spinner ABC123-login-spinner

Does it look OK? (y/n): y
```

**`work 123`** when exactly one issue directory matches (wrapper ends up **`cd`**-ing into that tree):

```text
(silent — your prompt moves to BRANCHES_PATH/ABC123-login-spinner)
```

**`work details`** (from a branch whose name contains the issue id; uses **`JIRA_BASE`**):

```text
Opening JIRA issue: ABC123
URL: https://jira.example.com/browse/ABC123
```

**`work push`** (fabricated — real output is plain **`git push`**):

```text
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
...
To https://github.com/acme/app.git
   abc1111..def2222  ABC123-login-spinner -> ABC123-login-spinner
```

## What this does (and does not)

- **Bash-only**: The `work` script targets Bash. `work init` appends a `work()` function to **`~/.bashrc`** (or **`WORK_BASHRC`** in `.env`).
- **Requires local config**: You must create `.env` beside the `work` script (from `.env.template`). Nothing runs without it.
- **Git conventions**: Listing and checkout flows assume a typical **`origin`** remote layout (`remotes/origin/...`). Adjust your workflow or script if you use different remote names.
- **Not a general Git UI**: This is a small helper for worktrees and issue-style branch names, not a replacement for `git` itself.

## Shell setup

**After `work init`**, a `work()` function is appended to **`~/.bashrc`** (or **`WORK_BASHRC`** in `.env`). It runs the `work` script from this checkout and applies the post-command `cd` behavior.

Open a new terminal or run:

```bash
source ~/.bashrc
```

Until you run init, use **`./work`** from the clone.

### First-time from a fresh clone

1. Copy `.env.template` to `.env` next to `work` and fill in **GIT_REPO**, **BRANCHES_PATH**, **MAIN_PATH**, and **PROJ_PREFIX** or **PROJ_PREFIXES** (see [`work` configuration](#work-configuration)). Add **JIRA_BASE** if you will use `work details`.
2. Run `./work init` — creates `BRANCHES_PATH`, clones into `MAIN_PATH` only when that path is missing or an empty directory (skips clone if a repo is already there), and installs or refreshes the **`work()`** wrapper in your bashrc (safe to re-run; re-run after moving this repo).

## Commands

Run **`work --help`** (or **`work -h`**) for the full command list and usage text.

## `work` configuration

Copy `.env.template` to `.env` in the same directory as the `work` script.

**Required:**

- **`GIT_REPO`** — Remote URL `work init` uses to clone the main repo into `MAIN_PATH` when that tree is missing.
- **`BRANCHES_PATH`** — Parent directory for worktrees created by `work checkout` / `work create`.
- **`MAIN_PATH`** — Path to the main clone (default git context for list, fetch, push, and related commands).
- **`PROJ_PREFIX`** or **`PROJ_PREFIXES`** — At least one must be set: use **PROJ_PREFIX** for a single issue key (e.g. `ABC`), or **PROJ_PREFIXES** for an ERE alternation (e.g. `ABC|DEF|GHI`) when you have multiple keys; if **PROJ_PREFIXES** is non-empty it is used for issue-id detection instead of **PROJ_PREFIX**.

**Optional (or needed for specific commands):**

- **`JIRA_BASE`** — Base URL for your issue tracker; `work details` appends the extracted issue id. Set this if you use `work details`.
- **`AUTO_CD_AFTER_CHECKOUT`** — When set to `true`, after a successful `work checkout` or `work create` the wrapper records the new worktree path so your shell `cd`s there automatically. If unset or not `true`, that automatic `cd` does not happen (post-worktree hooks still run when configured).
- **`WORK_MAIN_ALIAS`** — keyword for `work <keyword>` to jump to the main clone (`MAIN_PATH`). Default is `main`; set to `code`, `master`, etc. if you prefer.
- **`RELEASE_PREFIX`** — prefix used to identify release branches (default `release`). Used for the `RELEASES` list and for matching release branches in `work checkout`.
- **`WORK_POST_WORKTREE_DIR`** — directory of scripts to run after a successful `work checkout` or `work create`. Relative paths are resolved from **this git-work clone** (next to the `work` script); use an absolute path to point elsewhere. Default `.work/post-worktree.d` — commit that folder **here** so hooks follow your tooling clone. Executable files and `*.sh` files run in **sorted** order (e.g. `10-foo.sh`, `20-bar`). Environment: `WORK_WORKTREE_PATH`, `WORK_HOOK_REASON` (`checkout` or `create`). If the directory is missing, nothing runs. A failing hook is reported on stderr; the worktree is still created.
- **`WORK_BASHRC`** — file where `work init` appends the `work()` block (default `~/.bashrc`).

## Security note

The `work()` wrapper communicates the directory to `cd` into via a fixed file under **`/tmp/last_worktree_path_goto`**, which is then evaluated in the shell. On shared machines, `/tmp` symlink races are theoretically possible. Use this tool in environments you trust, or harden locally (for example by changing the script to use a private path under `$XDG_RUNTIME_DIR` or `$HOME`).

To report security issues privately, see [SECURITY.md](SECURITY.md).

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Publishing on GitHub (for maintainers)

Suggested repository **topics**: `git`, `worktree`, `bash`, `developer-tools`, `shell`. Set the repository **description** to the opening tagline under the title (or equivalent one-liner).
