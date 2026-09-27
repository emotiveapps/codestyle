# AGENT.md — agent guide for codestyle

This repo is the single source of truth for lint/format policy across all emotiveapps / Lucky Frog repos, organized as one top-level folder per language (currently `swift/`). `swift/.swiftlint.yml` is consumed **remotely** by every child repo via `parent_config:` — a merged change to `main` changes lint behavior everywhere on the next run. Treat every edit as a cross-repo policy change, not a local tweak.

## Rules for working in this repo

- **Don't loosen thresholds to silence a warning in one project.** The size/complexity rules are deliberately warning-only nags (owner's policy, Aug 2026); a project with a noisy file should fix or tolerate the warning, not weaken the shared policy. Only the owner decides threshold changes.
- **Keep the SwiftLint and SwiftFormat configs non-conflicting.** The owner's hard requirement: running `swiftformat .` then `swiftlint --fix` (in either order) must reach a fixed point — neither tool may undo the other. Known coupled pairs are documented at the top of `swift/.swiftformat` and in README. When touching either file, verify no enabled SwiftFormat rule rewrites code into a SwiftLint violation and vice versa (trailing commas, brace position, number grouping, whitespace are the classic fights).
- **`swift/.swiftlint-for-child-repos.yml` must stay minimal.** It's a template: `parent_config:` URL + universally-needed `excluded:` paths. Repo-specific config belongs in each repo's own `.swiftlint.yml`.
- **The repo must remain public** — SwiftLint fetches `parent_config` over unauthenticated HTTPS from the raw GitHub URL. Never move the canonical files or rename `main` without updating every child repo's `parent_config`.
- SwiftFormat is **nicklockwood/SwiftFormat** (CLI-flag config format), not Apple's swift-format (JSON `.swift-format`). Don't mix up their config syntaxes.
- New languages get their own top-level folder (`python/`, `typescript/`, …) following the same pattern: canonical config + child-repo template where the tool supports remote includes.

## Testing a change before pushing

Point a child repo's `parent_config:` at the local path temporarily, run its lint, then restore the URL:

```sh
# in the child repo's .swiftlint.yml (do not commit this)
parent_config: /path/to/codestyle/swift/.swiftlint.yml
```

For SwiftFormat: `swiftformat --lint --config /path/to/codestyle/swift/.swiftformat <child-repo>`.

Remember child repos cache the remote parent; after pushing a change, a stale cache can briefly mask it.

## Conventions

Changes go up as pull requests from `enderbot-author` for the owner to review and merge; nothing is pushed to `main` directly. Short imperative commit messages. A merged change reaches every child repo on its next lint run, so the PR says which child repos were linted against it.
