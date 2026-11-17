# Bluace agentic development guidelines for Business Central

Central source of the Claude Code rules and skills that Bluace uses for AL development.
Changes are made here once. Each AL project then pulls them in with the
`setup-al-vibe-rules` skill.

## Contents

| Path | Purpose |
|---|---|
| `.claude/rules/bluace-*.md` | Bluace team rules: coding standards, folder structure, compiling, testing, development environment. Copied into each project's `.claude/rules/`. |
| `.claude/skills/setup-al-vibe-rules/` | Skill that sets up or refreshes an AL project (see below). |
| `CLAUDE.md` | Reference `CLAUDE.md` for AL projects: working style and where Microsoft objects and source code come from. |

## What the skill does

Running `/setup-al-vibe-rules` in an AL project:

1. Downloads the official Microsoft AL rules (`al-*.md`) from
   [microsoft/alguidelines](https://github.com/microsoft/alguidelines) into `.claude/rules/`.
2. Updates the Bluace rules (`bluace-*.md`) from branch `main` of this repository.
3. Makes `.claude` the first folder in `al.code-workspace`.
4. Adds an `AL coding standards` section to `CLAUDE.md` that points to `.claude/rules/`.
5. Makes git ignore every `settings.local.json`.
6. Adds the AL MCP servers (`bc-knowledge`, `al`) to `.mcp.json`.
7. Enables the [bc-observatory](https://waldo1001.github.io/waldo.BCObservatory/) plugin for the
   project in `.claude/.claude/settings.json`. It connects `bc-observatory` (AL objects,
   versions, localizations), `bc-code-atlas` (AL source code) and `microsoft-learn`, and replaces
   the local copy of the Microsoft source (`external/BCApps`) that projects used before.
8. Creates `.claude/.claude/settings.local.json` with placeholders for the login of the local
   BC container.

For both rule sets, it first removes that set's old files, so rules that were removed or
renamed upstream disappear from the project too. Other files in `.claude/rules/` are left
alone. The skill never commits: review the changes and commit them with the project.

## Getting started

Requirements:
- Windows with PowerShell. The skill's download and copy commands are written for PowerShell.
- Git. The skill clones this repository with `git clone`; it is public, so no GitHub account or
  login is needed.
- Node.js 20 or later (for `npx`) and the AL extension in VS Code, for the MCP servers.
- The [ALCops](https://marketplace.visualstudio.com/items?itemName=arthurvdv.alcops) extension in
  VS Code. It provides the best-practice analyzers (`ALCops.*.dll`) that the agent passes to
  `alc.exe` (see `bluace-compile.md`) and fixes the diagnostics of after compiling. The Bluace
  ruleset in each app's `al.ruleSetPath` switches off the rules the team does not use.
- BC Code Intelligence, the `bc-knowledge` MCP server (npm package `bc-code-intelligence-mcp`).
  The skill registers it in `.mcp.json` and Claude Code starts it with `npx`, so the npm registry
  must be reachable. See
  [BC Code Intelligence](https://marketplace.visualstudio.com/items?itemName=jeremyvyska.bc-code-intelligence).
- The BC Observatory plugin for Claude Code. It provides the Microsoft objects, source code and
  documentation (`bc-observatory`, `bc-code-atlas` and `microsoft-learn`); the rules expect it.
  Install it as described in the
  [BC Observatory installation instructions](https://waldo1001.github.io/waldo.BCObservatory/mcp/#install).
  The skill also enables it in each project's `.claude/.claude/settings.json`. That registers the
  marketplace once you trust the project folder, but does not install the plugin: if you have not
  installed it yet, run `claude plugin install bc-observatory@bc-observatory --scope project`
  ([Require plugins per repository](https://code.claude.com/docs/en/plugins/org)).

Install the skill once, as a personal skill, so it is available in every project:

```powershell
git clone https://github.com/BluaceBV/bcdevguidelines.git C:\Repositories\bcdevguidelines
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force C:\Repositories\bcdevguidelines\.claude\skills\setup-al-vibe-rules "$HOME\.claude\skills\"
```

Then, in an AL project:

1. Open the project and start Claude Code.
2. Run `/setup-al-vibe-rules`. Claude does not start the skill on its own: you have to
   invoke it.
3. Fill in the login of your local container in `.claude/.claude/settings.local.json`.
4. Restart Claude Code, approve the project's MCP servers, accept the installation of the
   bc-observatory plugin and commit the changes.

Run the skill again in a project to bring its rules up to date.

## Changing rules or skills

1. Edit the rule files in `.claude/rules/` or the skill in this repository.
2. Commit and push to `main`. Projects only receive what is on `main`.
3. If the skill itself changed, copy it to `~\.claude\skills\` again (see Getting started).
4. Run `/setup-al-vibe-rules` in the projects that need the change.

Guidelines for rule files:
- Name team rules `bluace-<topic>.md`. Files with the `al-` prefix are overwritten by the
  Microsoft rules.
- Keep each rule focused on one topic. Explain the intent and include examples.
- Do not put credentials or customer-specific details in this repository.
