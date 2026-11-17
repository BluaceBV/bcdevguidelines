# Bluace: Project Context

The `bluace-*.md` rules are shared between repositories and contain no project-specific names, servers, companies or suffixes. Everything project-specific is read from the project files, so the rules can be copied to another repository without changes.

## Rule 1: Where project facts come from

### Intent
Before building, publishing, testing or naming objects, determine these facts for the current repository. Use the first source that has the answer.

| Fact | Source |
|---|---|
| Apps | Every folder in the repository with an `app.json` (normally the folders in `al.code-workspace`) |
| Test apps | Apps whose `app.json` depends on a test framework app: "Library Assert", "Tests-TestLibraries", "Test Runner" or "Any". All other apps are application apps |
| Build setup | AL-Go when the repository has a `.AL-Go` folder; otherwise the older setup with `Initialize.ps1`, `settings.json` and `scripts/` |
| App name, publisher, version, ID range | `app.json` of the app |
| Target BC version | `application` and `platform` in `app.json` |
| Object name suffix (affix) | `mandatorySuffix` in `AppSourceCop.json` of the app being changed (see Rule 2); otherwise `CRS.ObjectNameSuffix` in that app's `.vscode/settings.json`; otherwise the common suffix of its existing object names |
| Analyzers and ruleset | `al.codeAnalyzers` and `al.ruleSetPath` in the app's `.vscode/settings.json` |
| Development environment | The first configuration with a `server` value in the `.vscode/launch.json` of the app being built, published or tested (see `bluace-dev-environment.md`, Rule 1). An online sandbox only when it is the only configuration in that `launch.json` and the user approved it in the session (Rule 2) |
| Company | Not specified: publishing needs no company, and tests run in the default company of the local environment (the company it starts in) |
| Dependency order between apps | `dependencies` in each `app.json` |

If a fact cannot be determined, ask instead of guessing. Never take a server, company or suffix from an example in these rules.

## Rule 2: The suffix comes from the app being changed

### Intent
Suffixes differ per app, also between apps in the same repository. Always read the suffix of the app that contains the file being changed, from `mandatorySuffix` in that app's `AppSourceCop.json`, never from another app.

The value usually starts with a space (`" ABC"`). Trim it: the suffix is `ABC`.
- Object names: the name, a space, the suffix: `"Payment Link Helper ABC"`.
- Members of table and page extensions (procedures, global variables, actions): the suffix directly after the name, without a space: `CheckBalAccountNotUsedABC`.
- Variables and parameters in the app's own objects (not extension objects) whose type is an object of the same app: no suffix, also not the one in the object name: `PaymentLink: Record "Payment Link ABC"`, `PaymentLinkHelper: Codeunit "Payment Link Helper ABC"`.
- Variables and parameters whose type is an object of another app, such as an object of the app under test in a test app: the object name including its suffix, without spaces: `PaymentLinkABC: Record "Payment Link ABC"`.
- Variables and parameters in extension objects keep the suffix, like the other members of an extension.

If `AppSourceCop.json` has `mandatoryPrefix` instead, use that as a prefix in the same way. The file is only read for the affix; AppSourceCop is not used as an analyzer (see `bluace-compile.md`).

## Rule 3: Examples use a fictitious app

### Intent
The examples in the `bluace-*.md` rules use a fictitious app "Example App" with object name suffix `ABC`, and a test app "Example App.Test". Replace `ABC` with the suffix of the current app, and the example objects with the app's own objects. The patterns are what matter, not the names.
