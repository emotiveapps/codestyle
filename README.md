# codestyle

Shared lint and format configuration for all emotiveapps / Lucky Frog projects, organized by language. One place to change policy; every repo picks it up.

## What's here

| File | Role |
| --- | --- |
| `swift/.swiftlint.yml` | Canonical SwiftLint config. Child repos consume it **remotely** — do not copy it around. |
| `swift/.swiftlint-for-child-repos.yml` | Template for child repos: copy into a repo as `.swiftlint.yml`. It points `parent_config:` at the raw GitHub URL of the canonical config and holds only repo-specific overrides. |
| `swift/.swiftformat` | Canonical SwiftFormat ([nicklockwood/SwiftFormat](https://github.com/nicklockwood/SwiftFormat)) config. SwiftFormat has **no remote include**, so copy this file into each repo root (or run `swiftformat --config <path-to-this-repo>/swift/.swiftformat`). |

Other languages get their own top-level folder as the need arises.

## Setting up a new Swift repo

```sh
cd ~/Development/<the-repo>
curl -fsSL https://raw.githubusercontent.com/emotiveapps/codestyle/main/swift/.swiftlint-for-child-repos.yml -o .swiftlint.yml
curl -fsSL https://raw.githubusercontent.com/emotiveapps/codestyle/main/swift/.swiftformat -o .swiftformat
```

Add repo-specific `excluded:` paths below the template's globs, or opt-in rules, in the local `.swiftlint.yml`; local keys override the parent. Leave shared thresholds alone — change those here instead.

Exclusions must live in the child. A parent's `excluded:` paths resolve against the parent's own location, so they never reach a child repo; that is why the template carries `**/.build`, `**/.build-linux`, `**/.swiftpm` and `**/Derived` and the canonical config carries none. A local key replaces the parent's whole entry for that rule, so a child that sets `identifier_name` must repeat the parent's values it still wants.

## How the remote parent works

SwiftLint fetches `parent_config` over HTTPS on the first run and caches it (per-config, in the local cache dir). Runs after that are offline-safe; a changed canonical config is re-fetched automatically. This repo must stay **public** for the raw URL to resolve without auth.

## Swift policy (Aug 2026)

- Size/complexity rules (function/type/file length, cyclomatic complexity, large tuples) are **warning-only**: they nag at sensible thresholds but never fail a build.
- Errors are reserved for: identifiers shorter than 2 characters, identifiers that start uppercase, lines over 200 chars, and `#warning` messages missing a `TODO` prefix.
- Raw identifiers (SE-0451, Swift 6.2) are exempt from `identifier_name`, so a Swift Testing name can be the description: ``@Test func `Radius scale is ordered`()``.
- Line length warns at 140, and SwiftFormat wraps at 140.
- Trailing commas are kept in multiline lists and argument lists.
- `todo` stays on: every `TODO` comment warns.
- `statement_position` stays on: `} else {` and `} catch {` go on the closing brace's line. A `} else if` too long for 140 columns is split by hand at its conditions, not wrapped before `else`.
- `opening_brace` is disabled so long function signatures can wrap below the line limit.
- `multiline_arguments` is on: a call's arguments go all on one line, or one per line once wrapped.
- `inclusive_language` allows "master" for its meanings outside version control (a master key); branches are still `main`.

## SwiftLint ⇄ SwiftFormat compatibility

The two configs are maintained as a pair so `swiftformat .` and `swiftlint --fix` never fight:

- SwiftFormat's `trailingCommas` is enabled and SwiftLint's `trailing_comma` is disabled, so the commas SwiftFormat adds stay.
- `--maxwidth 140` matches SwiftLint's `line_length` warning.
- SwiftFormat's `elseOnSameLine` agrees with SwiftLint's `statement_position`. The one place they can disagree is a `} else if` line past 140 columns, which SwiftFormat's wrap may break before `else`; keep such conditions short enough, or split them across lines after `else if`.
- `wrapMultilineStatementBraces` is disabled to match SwiftLint's disabled `opening_brace`.
- `--decimalgrouping 3,6` matches SwiftLint's `number_separator` (separators required from 6 digits).
- `--trimwhitespace` agrees with SwiftLint's `trailing_whitespace`.
- `--wrap-arguments before-first` with `--allow-partial-wrapping false` does the rewriting that SwiftLint's opt-in `multiline_arguments` checks: a call's arguments go all on one line, or one per line once wrapped. `--wrap-parameters preserve` keeps declarations out of it.

If you change either file, re-check this table — see `AGENT.md`.
