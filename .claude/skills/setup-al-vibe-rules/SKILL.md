---
name: setup-al-vibe-rules
description: Bootstrap an AL project with the official Microsoft AL vibe coding rules
  from the alguidelines GitHub repo and the Bluace team rules from the bcdevguidelines
  GitHub repo. Downloads all rule files to .claude/rules/, adds
  .claude/ to the workspace file, updates CLAUDE.md with the location of these rules,
  excludes settings.local.json from git, creates or updates .mcp.json with the AL MCP servers,
  enables the bc-observatory plugin (Microsoft objects and source code through bc-observatory,
  bc-code-atlas and Microsoft Learn) for the project and creates the local settings file for
  the AL MCP login.
  Use when starting a new AL project, onboarding a new developer, or refreshing stale rule files.
disable-model-invocation: true
---

# Setup AL Vibe Coding Rules

Fetches the official Microsoft AL coding rules and the Bluace team rules from GitHub and wires them into the
VSCode project so every future agent session follows them automatically. It also registers
the MCP servers used for AL development in .mcp.json, enables the bc-observatory plugin that
provides the Microsoft objects and source code (see `bluace-microsoft-source.md`), and makes
sure the local settings file with the AL MCP login exists and can never be committed.

There is no local copy of the Microsoft source (such as an `external/BCApps` folder) anymore:
the plugin's MCP servers replace it.

All paths below are relative to the project directory: the folder that holds CLAUDE.md,
al.code-workspace and .claude/. Determine it first from the current working directory:
- if the name of the current folder is `.claude`, the project directory is its parent folder
  (in VS Code with al.code-workspace the session usually starts in the `.claude` folder);
- otherwise the project directory is the current folder.

Use absolute paths based on the project directory in every step, and run git commands with
`git -C <project directory>`, so the result does not depend on where the session started.

## Step 1 - Discover files in the GitHub folder

Call the GitHub Contents API to get the current file list (never hard-code filenames):

GET https://api.github.com/repos/microsoft/alguidelines/contents/content/docs/agentic-coding/vibe-coding-rules

Parse the JSON response and collect every entry where "type": "file" and the name
matches al-*.md. Skip README.md, _index.md and any other non-rule files.
Extract the name and download_url of each file.

## Step 2 - Download each file directly into .claude/rules/

Delete only the existing `al-*.md` files in .claude/rules/ (or create the folder if missing),
so rules that were removed or renamed upstream do not linger. Never delete other files:
the folder also holds team rules such as `bluace-*.md`. Then download every file from its
download_url into .claude/rules/<filename>.
Do NOT use WebFetch and then Write - download straight to disk.

On Windows (PowerShell), run all downloads in one shell call:

New-Item -ItemType Directory -Force -Path "<project directory>/.claude/rules" | Out-Null
Get-ChildItem "<project directory>/.claude/rules" -File -Filter 'al-*.md' | Remove-Item -Force
Invoke-WebRequest -Uri "<download_url>" -OutFile "<project directory>/.claude/rules/<filename>"

Chain multiple downloads with ; so every file is saved in a single round-trip.

## Step 3 - Update the Bluace rules from bcdevguidelines

The Bluace team rules (`bluace-*.md`) come from the folder `.claude/rules/` on branch `main`
of https://github.com/BluaceBV/bcdevguidelines. That repository is private: the GitHub API and
raw download URLs return 404 without a login. Use `git clone` instead, which logs in with the
developer's own GitHub credentials.

Skip this step if the project directory is the bcdevguidelines repository itself (its
`git remote get-url origin` is the URL above): it is the source of these rules.

Clone to a temporary folder first and only replace the rules when the clone succeeded and
contains rules, so a failed clone never leaves the project without Bluace rules. Remove the
existing `bluace-*.md` files before copying, so rules that were removed or renamed upstream
do not linger. Never touch other files in .claude/rules/.

On Windows (PowerShell), run it in one shell call:

$rules = "<project directory>/.claude/rules"
$src = Join-Path ([IO.Path]::GetTempPath()) ("bcdevguidelines-" + [guid]::NewGuid())
git clone --depth 1 --branch main --quiet https://github.com/BluaceBV/bcdevguidelines.git $src
$new = if ($LASTEXITCODE -eq 0) { Get-ChildItem "$src/.claude/rules" -File -Filter 'bluace-*.md' -ErrorAction SilentlyContinue }
if ($new) {
    New-Item -ItemType Directory -Force -Path $rules | Out-Null
    Get-ChildItem $rules -File -Filter 'bluace-*.md' | Remove-Item -Force
    Copy-Item $new.FullName -Destination $rules
    $new.Name
} else { Write-Warning "No Bluace rules retrieved; existing bluace-*.md files left unchanged." }
if (Test-Path $src) { Remove-Item -Recurse -Force $src }

If no rules were retrieved, continue with the other steps and tell the user at the end. The
usual cause is that git is not logged in to GitHub with an account that has access to the
BluaceBV organisation.

## Step 4 - Add .claude/ to the workspace file

Read al.code-workspace and make sure { "path": ".claude" } is the first entry of the
folders array: add it if missing, move it to the front if it is elsewhere. The Claude Code
session starts in the first workspace folder, and steps 8 and 9 rely on that. Keep the other
entries and the existing formatting intact.

## Step 5 - Add the AL coding standards section to CLAUDE.md

Append the following section to CLAUDE.md. Skip this step if CLAUDE.md already contains
an `AL coding standards` heading at any level (`#`, `##`, ...).

```markdown
## AL coding standards
- See .claude/rules/ for detailed rules.
```

Then check CLAUDE.md for instructions that point to a local copy of the Microsoft source, such
as lines that mention `external/BCApps` or its `src/Layers/...` folders (older projects copied
them from the reference CLAUDE.md of bcdevguidelines). That folder is no longer used:
`bluace-microsoft-source.md` replaces those instructions. Show the user the lines and ask before
replacing them with this one line, keeping the rest of CLAUDE.md unchanged:

```markdown
- Microsoft objects and source code come from the bc-observatory plugin (bc-observatory, bc-code-atlas, microsoft-learn), not from a local copy of BCApps. See .claude/rules/bluace-microsoft-source.md.
```

If the project directory has an `external/BCApps` folder or a git submodule for it, mention it,
but do not remove it: leave that to the user.

## Step 6 - Exclude settings.local.json from git

Every `settings.local.json` holds local settings such as the AL MCP login and must never be
committed. Do this before step 9, so the file is ignored before it exists.

1. Make sure .gitignore in the project directory contains this section (create the file if missing):

```gitignore
# Local Claude Code settings (may contain credentials)
**/settings.local.json
```

   Skip this if .gitignore already has a line that is exactly `**/settings.local.json` or
   `settings.local.json`. A narrower pattern such as `**/.claude/settings.local.json` does not
   count: it misses files in other folders. Otherwise append the two lines at the end, after
   making sure the existing last line ends with a newline and leaving one empty line before
   the new section. Do not change any other line.

2. `.gitignore` does not affect files that git already tracks. Run
   `git -C <project directory> ls-files -- '*settings.local.json'`. If it lists files, tell the
   user which ones, and ask before running `git -C <project directory> rm --cached -- <file>` for each (this keeps the file on disk but
   removes it from the repository after the next commit). Warn that the content stays in the
   git history: a password or token that was ever pushed must be changed.

## Step 7 - Create or update .mcp.json

.mcp.json in the project directory registers the MCP servers for AL development for everyone
who works in the repository. It contains no credentials (the AL MCP login comes from step 9),
so it is committed.

These servers belong in it:

```json
{
  "mcpServers": {
    "bc-knowledge": {
      "command": "npx",
      "args": [
        "bc-code-intelligence-mcp"
      ]
    },
    "al": {
      "command": "al",
      "args": [
        "launchmcpserver",
        "--transport",
        "stdio"
      ]
    }
  }
}
```

The servers `bc-observatory`, `bc-code-atlas` and `microsoft-learn` are not added here: the
bc-observatory plugin of step 8 provides them.

- If the file does not exist, create it with exactly this content.
- If it exists, add each server above whose name is missing from `mcpServers` (add the
  `mcpServers` object if it is missing). Do not change or remove existing servers, also not
  when a server with the same name has a different configuration: report that difference to
  the user instead. Keep the existing formatting and the order of the existing entries.
- If the file has a server for Microsoft Learn (e.g. `microsoft-learn` with the url
  `https://learn.microsoft.com/api/mcp`, added by an earlier version of this skill), or for
  `bc-code-atlas` or `bc-observatory`, tell the user that the plugin of step 8 provides it, and
  ask before removing it: otherwise its tools appear twice.
- Never put credentials or tokens in .mcp.json; it is shared through git.

## Step 8 - Enable the bc-observatory plugin for the project

The Microsoft objects and source code are looked up through the bc-observatory plugin (see
`bluace-microsoft-source.md`). It connects the MCP servers `bc-observatory` (facts about AL
objects, versions and localizations), `bc-code-atlas` (AL source code) and `microsoft-learn`
(documentation). Enabling it in the project settings makes the requirement visible to everyone
who works in the repository. It does not install the plugin: Claude Code registers the
marketplace after the developer accepts the workspace trust dialog (interactive sessions only),
and each developer installs the plugin once, if they have not done so yet
(see https://code.claude.com/docs/en/plugins/org, "Require plugins per repository").

Project settings are read from the `.claude` folder of the folder where the session starts,
which is the `.claude` workspace folder (step 4). The file is therefore
`<project directory>/.claude/.claude/settings.json`, next to the `settings.local.json` of step 9.
It contains no credentials and is committed.

These settings belong in it:

```json
{
  "extraKnownMarketplaces": {
    "bc-observatory": {
      "source": {
        "source": "url",
        "url": "https://waldo1001.github.io/waldo.BCObservatory/marketplace.json"
      }
    }
  },
  "enabledPlugins": {
    "bc-observatory@bc-observatory": true
  }
}
```

- If the file does not exist, create it with exactly this content.
- If it exists, add the `bc-observatory` entry to `extraKnownMarketplaces` and the
  `bc-observatory@bc-observatory` entry to `enabledPlugins` when they are missing (add the
  objects themselves when missing). Keep all other content unchanged. If an entry exists with a
  different value (e.g. the plugin set to `false`), report it to the user instead of changing it.
- Never put credentials in this file; it is shared through git.

## Step 9 - Create the local settings file

The AL MCP tools log in to the local container through `BC_SERVER_USERNAME` and
`BC_SERVER_PASSWORD` (see `bluace-dev-environment.md`). They are read from the
`settings.local.json` in the `.claude` folder of the folder where the session starts. Sessions
are normally started from the workspace, whose first folder is `.claude`, so the file is always
`<project directory>/.claude/.claude/settings.local.json`, wherever this skill runs.

Its path relative to the current folder depends on where the session started:
- started from the workspace (current folder is `.claude`): `.claude/settings.local.json`;
- started from the project directory: `.claude/.claude/settings.local.json`.

Do not create `<project directory>/.claude/settings.local.json` for the login: a session
started from the workspace does not read that file.

- If the file does not exist, create it with:

```json
{
  "env": {
    "BC_SERVER_USERNAME": "<username>",
    "BC_SERVER_PASSWORD": "<password>"
  }
}
```

- If it exists, do not overwrite it. Add an `env` block or the missing variables with the
  placeholder values above, and keep all other content and existing values unchanged.
- Never ask for, show or log the password. The user fills in the placeholders themselves.

## Verification

After completing all steps, confirm:
- .claude/rules/ contains the al-*.md files from the GitHub folder, and the other rule files are still there
- .claude/rules/ contains exactly the bluace-*.md files from bcdevguidelines (unless step 3 was skipped or failed), and the temporary clone is removed
- al.code-workspace has { "path": ".claude" } as the first entry in folders
- CLAUDE.md contains the `AL coding standards` section exactly once
- every settings.local.json in the project directory is ignored: `cd <project directory> && find . -name settings.local.json -not -path './.git/*' -exec git check-ignore -v {} \;` reports each file with the `**/settings.local.json` line in .gitignore
- CLAUDE.md has no instructions left that point to `external/BCApps` (unless the user chose to keep them)
- .mcp.json is valid JSON and contains the servers `bc-knowledge` and `al`
- .claude/.claude/settings.json is valid JSON, has the `bc-observatory` marketplace and enables `bc-observatory@bc-observatory`
- .claude/.claude/settings.local.json exists with both variables

Finish by telling the user to replace the placeholders in .claude/.claude/settings.local.json
with the login of their local container (if they are still placeholders), and to restart
Claude Code afterwards, because settings and MCP servers are only read when a session starts.
If the skill ran in a session started from the project directory, add that the login is only
picked up by sessions started from the workspace (al.code-workspace).
If .mcp.json was created or got new servers, tell them that Claude Code asks to approve the
project's MCP servers on the next start.
If step 8 created or changed .claude/.claude/settings.json, tell them that enabling the plugin
in the project does not install it. If `claude plugin list` does not show
`bc-observatory@bc-observatory`, they install it as described in
https://waldo1001.github.io/waldo.BCObservatory/mcp/#install, or with
`claude plugin install bc-observatory@bc-observatory --scope project` after accepting the trust
dialog for the folder. The plugin needs Node.js (`npx`).
If step 3 retrieved no Bluace rules, tell them and that the existing bluace-*.md files were left unchanged.

## Notes
- Always fetch from GitHub - never generate or paraphrase rule content.
- CLAUDE.md should be kept short; detailed rules live in .claude/rules/.
- Do not commit; leave that to the user.
