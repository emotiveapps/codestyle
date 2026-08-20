# swift-style

Shared Swift lint and format configuration for all emotiveapps / Lucky Frog projects. One place to change policy; every repo picks it up.

## What's here

| File | Role |
| --- | --- |
| `.swiftlint.yml` | Canonical SwiftLint config. Child repos consume it **remotely** — do not copy it around. |
| `.swiftlint-for-child-repos.yml` | Template for child repos: copy into a repo as `.swiftlint.yml`. It points `parent_config:` at the raw GitHub URL of the canonical config and holds only repo-specific overrides. |
| `.swiftformat` | Canonical SwiftFormat ([nicklockwood/SwiftFormat](https://github.com/nicklockwood/SwiftFormat)) config. SwiftFormat has **no remote include**, so copy this file into each repo root (or run `swiftformat --config <path-to-this-repo>/.swiftformat`). |

## Setting up a new repo

```sh
cd ~/Development/<the-repo>
curl -fsSL https://raw.githubusercontent.com/emotiveapps/swift-style/main/.swiftlint-for-child-repos.yml -o .swiftlint.yml
curl -fsSL https://raw.githubusercontent.com/emotiveapps/swift-style/main/.swiftformat -o .swiftformat
```

Add repo-specific `excluded:` paths or opt-in rules to the local `.swiftlint.yml`; local keys override the parent. Leave shared thresholds alone — change those here instead.

## How the remote parent works

SwiftLint fetches `parent_config` over HTTPS on the first run and caches it (per-config, in the local cache dir). Runs after that are offline-safe; a changed canonical config is re-fetched automatically. This repo must stay **public** for the raw URL to resolve without auth.

## Policy (Aug 2026)

- Size/complexity rules (function/type/file length, cyclomatic complexity, large tuples) are **warning-only**: they nag at sensible thresholds but never fail a build.
- Errors are reserved for: identifiers shorter than 2 characters, lines over 200 chars, and `#warning` messages missing a `TODO` prefix.
- Line length warns at 140.
- `opening_brace` is disabled so long function signatures can wrap below the line limit.

## SwiftLint ⇄ SwiftFormat compatibility

The two configs are maintained as a pair so `swiftformat .` and `swiftlint --fix` never fight:

- SwiftFormat's `trailingCommas` is disabled (it would add commas that SwiftLint's `trailing_comma` removes).
- `wrapMultilineStatementBraces` is disabled to match SwiftLint's disabled `opening_brace`.
- `--decimalgrouping 3,6` matches SwiftLint's `number_separator` (separators required from 6 digits).
- `--trimwhitespace` agrees with SwiftLint's `trailing_whitespace`.

If you change either file, re-check this table — see `AGENT.md`.
