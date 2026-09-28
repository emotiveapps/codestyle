# SwiftLint rules for Vapor server code

There is no separate central config for servers. A Vapor server gets the canonical rules the same way every Swift repo does, through `.swiftlint-for-child-repos.yml`, and adds the rules below in its own config. They are recommended for every Vapor project and are copied in, not fetched.

## Where they go

**A repo that is only a Vapor server:** add the rules to the repo's `.swiftlint.yml`, below the template's `parent_config:` and `excluded:`.

**A monorepo whose server sits beside apps** (LilyPad's `Billund/`, Minyana's `Minyana/Server`): put the rules in a `.swiftlint.yml` inside the server's folder, and give that file **no `parent_config:`**.

```
LilyPad/
├── .swiftlint.yml      the child template: parent_config is the canonical config
├── Apps/
└── Billund/
    └── .swiftlint.yml  only the server rules below; no parent_config
```

SwiftLint calls a `.swiftlint.yml` in a subfolder a *nested configuration*. When it runs from the repo root, it applies the root config to every file, and for files under `Billund/` it also applies the nested file's rules. So the server gets the canonical rules plus its own, and the apps get only the canonical ones.

A nested file cannot bring in a config of its own: SwiftLint's README says "`parent_config`/`child_config` specifications of nested configurations are getting ignored". A `parent_config:` there is skipped without a warning, which is why the server rules are written into the nested file itself rather than kept in a shared file it points at.

Two consequences:

- **Lint from the repo root.** Run from inside `Billund/`, the nested file becomes the only config, and the canonical rules are left out.
- **Don't exclude the server folder in the root config.** The root config has to reach those files for the canonical rules to apply to them.

## The rules

### `vapor_log_call_form`

Server code logs with the area first, `logger.<level>(in: .area, message: "...")`, as the apps do through Logbook. Vapor hands out swift-log's `Logger` as `req.logger` and `context.logger`, and swift-log's own `logger.info("...")` still compiles beside the area form, so the compiler cannot insist on it. This rule does. It catches a call that ends its line without `message:`; a call closed mid-line, inside a closure, slips past it.

```yaml
custom_rules:
  vapor_log_call_form:
    name: "Vapor log call form"
    regex: '\blogger\??\s*\.(?:trace|debug|info|notice|warning|error|critical)\((?:(?!message:)[\s\S])*?\)[ \t]*$'
    message: "Log with the area first: logger.<level>(in: .area, message: \"...\")."
    severity: error
```

Custom rules in a nested config are added to the root config's, not substituted for them, so the canonical `warnings_need_todo_prefix` still applies under the server folder.

A rule that belongs in every Vapor project is added here, and each server copies it in.
