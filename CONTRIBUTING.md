# Contributing

Thank you for helping improve git-work.

## Development

- The main script is [`work`](work). Configuration lives in [`.env.template`](.env.template) (copy to `.env` locally; `.env` is gitignored).
- Run **ShellCheck** before submitting changes:

  ```bash
  shellcheck work
  ```

  CI runs the same check on pull requests.

- Match existing Bash style (functions, `printf`, `[[ ... ]]`) and keep changes focused on the problem you are solving.

## Pull requests

- Describe the behavior change and why it is needed.
- If you change user-visible behavior or configuration, update [`README.md`](README.md) in the same PR.

## Security-sensitive changes

- Do not commit real `GIT_REPO` URLs, tokens, or internal hostnames into tracked files or commit messages.
- If you ever need to remove sensitive data from history before a repository is public, use history rewriting (for example `git filter-repo`) and coordinate with repository owners.

## Pre–open-source history check (maintainers)

Before the repository was made public, history was scanned for obvious secret patterns (tokens, private keys, common API key formats) with no matches and no history rewrite required. Re-run a similar check before any release if contributors might have committed local paths or credentials by mistake.
