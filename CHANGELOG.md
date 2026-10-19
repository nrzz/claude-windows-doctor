# Changelog

All notable changes to claude-windows-doctor are written here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

- `check --ignore <id,...>` leaves the named findings out of the report, the exit code and the JSON (`ignored: [...]`). Finding ids are comma-separated and case-insensitive, with surrounding whitespace ignored. When an unknown id is given, it exits 2 with a suggestion when close to an existing finding id. Giving `--ignore` more than once or with an empty list is a usage error (exit 2).
- Contributed by [@yoreva-01](https://github.com/yoreva-01) for the good first issue [#13](https://github.com/nrzz/claude-windows-doctor/issues/13).

## [1.5.0] - 2026-10-19

- `check --markdown --max <n>` sets how many errors and warnings the short report holds (12 without it, the skill's cap; 1 or more): `--max 3` prints three and says how many more there are, `--max 40` prints up to 40. `--max` without `--markdown` is a usage error (`--max goes with --markdown`), and anything but a whole number of 1 or more is refused with `--max needs a whole number, 1 or more` (exit 2). The skill is unchanged.
- Contributed by [@yoreva-01](https://github.com/yoreva-01) in [#12](https://github.com/nrzz/claude-windows-doctor/pull/12), for the good first issue [#11](https://github.com/nrzz/claude-windows-doctor/issues/11).

## [1.4.0] - 2026-10-19

- `explain <id> --markdown` prints a finding page as Markdown: a `## <id>: <title>` heading, the paragraph, "What Claude Code does" with each line's mark as `*(verified)*`, `*(documented)*` or `*(not verified)*`, the fix, and the issue threads as links (or `**The issue threads:** none found for this`). `explain --list --markdown`, with or without a part, prints the list as a `| id | title |` table. `--markdown` with `--json` is a usage error (exit 2); an unknown id or an ambiguous part is refused in text as before.
- Contributed by [@yoreva-01](https://github.com/yoreva-01) in [#10](https://github.com/nrzz/claude-windows-doctor/pull/10), for the good first issue [#9](https://github.com/nrzz/claude-windows-doctor/issues/9).

## [1.3.0] - 2026-10-19

- `explain --list <part>` prints the finding ids that contain the part, with their titles, in page order, in any case and with surrounding whitespace ignored (`explain --list git-bash` prints the four); `--json` gives the same document as `explain --list --json`, filtered. A part that no id contains exits 1 with `no finding id contains "<part>"` and, in JSON, every id and the nearest ids. `explain --list` with a part was a usage error before.
- Contributed by [@yoreva-01](https://github.com/yoreva-01) in [#8](https://github.com/nrzz/claude-windows-doctor/pull/8), for the good first issue [#7](https://github.com/nrzz/claude-windows-doctor/issues/7).

## [1.2.1] - 2026-10-18

- `explain` tells an ambiguous part from an unknown id: a part that several ids contain is refused with every id that contains it, in page order (`explain git-bash` names the four and says "Give one of them"), in text and in JSON (`ids` holds those ids and `near` is empty); an unknown id keeps its message with the nearest ids. Both exit 1.
- Contributed by [@Plastic-time](https://github.com/Plastic-time) in [#6](https://github.com/nrzz/claude-windows-doctor/pull/6), for the good first issue [#5](https://github.com/nrzz/claude-windows-doctor/issues/5).

## [1.2.0] - 2026-10-18

- `explain` takes a part of a finding id when exactly one id contains it: `explain long` prints `long-paths`, `explain distro` prints `wsl-no-distro`. A whole id still wins, and a part that several ids contain is refused like an unknown id, with the nearest ids (`explain git-bash` names the four). `--json` is unchanged: the page's `id` is the full id.
- Contributed by [@Doomguy-BFG9000](https://github.com/Doomguy-BFG9000) in [#4](https://github.com/nrzz/claude-windows-doctor/pull/4), for the good first issue [#3](https://github.com/nrzz/claude-windows-doctor/issues/3).

## [1.1.0] - 2026-10-17

- `explain --list` prints every finding id with its title, in the order of the pages, the id padded to the longest; `--json` gives `{ ok, findings: [ { id, title } ] }`. `explain` with neither an id nor `--list` is still a usage error (exit 2), and `--list` together with an id is one too.
- Contributed by [@Doomguy-BFG9000](https://github.com/Doomguy-BFG9000) in [#2](https://github.com/nrzz/claude-windows-doctor/pull/2), the first outside pull request, for the good first issue [#1](https://github.com/nrzz/claude-windows-doctor/issues/1).
- The real-binary check of the skill's hand-off accepts what Claude Code 2.1.302 does: a skill whose command is not allowed ends the turn with no request to the model, an empty result and `num_turns` 0, where 2.1.301 and earlier handed the model the command line as "run this first". The README records it.

## [1.0.0] - 2026-10-16

- First release. A CLI and a Claude Code plugin with one user-only skill, `/windows-doctor:check`.
- `check` says, on Windows, which shell tools Claude Code gives the model and why: the three places Claude Code looks for Git Bash in its order and what each answered, `CLAUDE_CODE_GIT_BASH_PATH`, `CLAUDE_CODE_USE_POWERSHELL_TOOL`, the whole-tool deny rules and `defaultShell` (which changes nothing in the model's tools), and the edition of PowerShell the tool runs. It lists every `claude` on PATH and in the places the installers use (the native installer, npm, WinGet, the desktop app's bundles, an editor's extension) with its version and which one a terminal runs first, and finds the setup problems behind common Windows issues: Node and npm, long paths, `core.autocrlf`, the PowerShell execution policy, the console code page, a stale `.git\index.lock`, `CLAUDE_*` and `ANTHROPIC_*` variables that are in the user environment and not in the terminal (or the other way round), and WSL. Inside a WSL distribution it checks the WSL side. On Linux and macOS it says there is nothing to check and exits 0. 27 findings, each with an id, a severity, what Claude Code does about it, and a fix.
- `check --live` proves the answer with the real `claude`: it starts the binary against a fake Messages API on 127.0.0.1 in a throwaway folder, reads the tools of the first request and the `bash.exe` the binary says it took, and compares them with the prediction. No token is spent.
- `check --markdown` is the short report of the skill: at most 12 findings, 500 characters a line, 16 lines; it always exits 0, because Claude Code drops a skill whose command fails.
- `fix --git-bash` sets `CLAUDE_CODE_GIT_BASH_PATH` in the Windows user environment for a Git Bash that Claude Code does not find, after recording what was there, and `undo` puts it back. Both take `--dry-run` and refuse off Windows. They change that one value and nothing else.
- `explain <finding-id>` prints what a finding means, what Claude Code does in that case (each line marked verified, documented or not verified), the fix and the issue threads.
- What Claude Code does is what the real binary did: `test/shell-table.mjs` holds 129 cases (fake Git trees, PATH orders, spellings of the two variables, settings files, deny rules, `defaultShell`) that were put to Claude Code 2.1.295 and 2.1.299 on Windows 11 and, in CI, to the newest release (2.1.301) on `windows-latest`, and the same table is held against the prediction on a pretend Windows machine on every system. Git Bash is looked for in `CLAUDE_CODE_GIT_BASH_PATH`, then `C:\Program Files\Git` and `C:\Program Files (x86)\Git`, then two folders up from the first `git.exe` that `where.exe` prints; a variable that points at a missing file is ignored and the search goes on.
- Zero tokens for the tool. The skill is user-only, adds nothing to a request that does not use it, and hands the model a report of at most 12 findings.

[1.5.0]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.5.0
[1.4.0]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.4.0
[1.3.0]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.3.0
[1.2.1]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.2.1
[1.2.0]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.2.0
[1.1.0]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.1.0
[1.0.0]: https://github.com/nrzz/claude-windows-doctor/releases/tag/v1.0.0

[Unreleased]: https://github.com/nrzz/claude-windows-doctor/compare/v1.5.0...HEAD
