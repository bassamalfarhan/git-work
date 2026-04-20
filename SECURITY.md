# Security policy

## Supported versions

Only the latest commit on the default branch is actively reviewed. There are no long-term release branches yet.

## Reporting a vulnerability

Please report security issues **privately** so they can be fixed before public disclosure.

- Use [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability) to report vulnerabilities privately for this repository.

Include steps to reproduce, affected component (for example the `work` script or `work init` bashrc integration), and impact if known.

## Known design considerations

The `work()` wrapper uses `/tmp/last_worktree_path_goto` to pass the next directory to `cd` into. See the [README](README.md#security-note) for context. Reports that harden this path without breaking the interactive flow are welcome.
