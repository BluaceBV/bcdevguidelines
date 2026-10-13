# Bluace: Local Development Environment

How to build and publish the apps to the local Business Central container with the AL MCP tools. The server, company and apps of the current repository come from `bluace-project-context.md`.

## Rule 1: The local container

### Intent
Always build, publish and test against the local container, never against a cloud sandbox or production environment.

Per app, use the first configuration in its `.vscode/launch.json` that has a `server` value (e.g. `http://mycontainer`): take `server`, `serverInstance`, `tenant` and `authentication` from it, and `port` when present (otherwise 7049). Configurations without `server` (such as those with only `environmentName` or a cloud `tenant`) are cloud environments: skip them. `launch.json` is gitignored and local to each developer, so it points to that developer's own container. If an app has no `launch.json` or no configuration with a `server`, ask instead of guessing.

How the container is created differs per repository:
- AL-Go repositories (with a `.AL-Go` folder): `.AL-Go/Bluace-DevEnv.ps1`, which calls AL-Go's `localDevEnv.ps1`. It compiles and publishes all apps in `appFolders` and `testFolders` of `.AL-Go/settings.json`.
- Older repositories (with `Initialize.ps1`, `settings.json` and `scripts/` in the root): `Initialize.ps1`, which uses the scripts in `scripts/` and adds a configuration to `launch.json`.

Do not create or remove containers unless asked.

## Rule 2: Test libraries come from AL-Go settings, not from container flags

### Intent
This applies to AL-Go repositories. AL-Go installs the test toolkit based on the test app's dependencies: a dependency on "Tests-TestLibraries" sets `installTestLibraries`; `"installTestLibraries": true` in `.AL-Go/settings.json` makes it explicit. Do not add test toolkit flags to `.AL-Go/NewBcContainer.ps1`: `$parameters.additionalParameters` are passed to `docker run`, so a flag such as `--includeTestLibrariesOnly` breaks container creation.

## Rule 3: Login for the AL MCP tools

### Intent
The AL MCP server authenticates with `UserPassword` only through the environment variables `BC_SERVER_USERNAME` and `BC_SERVER_PASSWORD`. `altool auth login` only supports Entra ID and cannot store an on-premises login.

- Put both variables in the `env` block of a `settings.local.json`. Never commit them: every `settings.local.json` must be excluded from git with `**/settings.local.json` in `.gitignore`; the skill `setup-al-vibe-rules` adds this and creates the file.
- The file must be in the `.claude` folder of the folder where the Claude Code session starts. In VS Code with `al.code-workspace`, the session starts in the first workspace folder, which is `.claude` itself, so the file is `.claude/.claude/settings.local.json`.
- Settings are read when the session starts: restart Claude Code after changing them.
- If the AL MCP tools report "Authentication required", first check that `$env:BC_SERVER_USERNAME` is set in the session.

## Rule 4: Building and publishing with the AL MCP tools

### Intent
- Add every app folder and test app folder with `al_addproject`.
- Build and publish one project at a time; `al_build` with `scope: all` only builds the first project.
- Publish in dependency order (from the `dependencies` in each `app.json`): an app before the apps that depend on it, test apps last.
- `al_publish` parameters: `serverUrl`, `serverInstance` and `port` of the local server, `authentication: UserPassword`, `environmentType: OnPrem`. To publish a package built with `alc.exe` (see `bluace-compile.md`), pass `appPath` with `skipBuild: true`.
- A dependent project needs the current symbols of its dependency: copy the freshly built `.app` into the dependent project's `.alpackages`, or download symbols from the container.
