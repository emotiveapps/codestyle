# codestyle

Shared lint and format configuration for all emotiveapps / Lucky Frog projects, organized by language. One place to change policy; every repo picks it up.

## What's here

| File | Role |
| --- | --- |
| `swift/.swiftlint.yml` | Canonical SwiftLint config. Child repos consume it **remotely** — do not copy it around. |
| `swift/.swiftlint-for-child-repos.yml` | Template for child repos: copy into a repo as `.swiftlint.yml`. It points `parent_config:` at the raw GitHub URL of the canonical config and holds only repo-specific overrides. |
| `swift/.swiftlint-vapor.yml` | SwiftLint for Vapor server packages: its parent is the canonical config, and it adds the rules every Vapor project uses. |
| `swift/.swiftlint-vapor-for-child-repos.yml` | Template for a Vapor package: copy into the package's folder as `.swiftlint.yml`. |
| `swift/.swiftformat` | Canonical SwiftFormat ([nicklockwood/SwiftFormat](https://github.com/nicklockwood/SwiftFormat)) config. SwiftFormat has **no remote include**, so copy this file into each repo root (or run `swiftformat --config <path-to-this-repo>/swift/.swiftformat`). |

Other languages get their own top-level folder as the need arises.

## Setting up a new Swift repo

```sh
cd ~/Development/<the-repo>
curl -fsSL https://raw.githubusercontent.com/emotiveapps/codestyle/main/swift/.swiftlint-for-child-repos.yml -o .swiftlint.yml
curl -fsSL https://raw.githubusercontent.com/emotiveapps/codestyle/main/swift/.swiftformat -o .swiftformat
```

Add repo-specific `excluded:` paths or opt-in rules to the local `.swiftlint.yml`; local keys override the parent. Leave shared thresholds alone — change those here instead.

## Setting up a Vapor package

```sh
cd ~/Development/<the-repo>/<the-server-package>
curl -fsSL https://raw.githubusercontent.com/emotiveapps/codestyle/main/swift/.swiftlint-vapor-for-child-repos.yml -o .swiftlint.yml
```

The chain is the package's `.swiftlint.yml`, then `swift/.swiftlint-vapor.yml`, then `swift/.swiftlint.yml`, so the package gets every canonical rule plus the Vapor ones. Custom rules add up along the chain rather than replacing each other.

Lint a Vapor package **from its own folder**. SwiftLint 0.65 honours a `.swiftlint.yml` in a subfolder when run from a repo root, but ignores that file's `parent_config`, so the Vapor rules would silently not apply. In a repo that also holds apps, add the package folder to the root `.swiftlint.yml`'s `excluded:` and run SwiftLint twice, once at the root and once in the package.

## How the remote parent works

SwiftLint fetches `parent_config` over HTTPS on the first run and caches it (per-config, in the local cache dir). Runs after that are offline-safe; a changed canonical config is re-fetched automatically. This repo must stay **public** for the raw URL to resolve without auth.

## Swift policy (Aug 2026)

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
