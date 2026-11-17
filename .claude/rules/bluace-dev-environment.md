# Bluace: Development Environment

How to build and publish the apps with the AL MCP tools: to the local Business Central container, or, only under the conditions of Rule 2, to an online (SaaS) sandbox. The server, company and apps of the current repository come from `bluace-project-context.md`.

## Rule 1: The local container is the default

### Intent
Build, publish and test against the local container. Use an online sandbox only under the conditions of Rule 2, and never use a production environment.

Per app, use the first configuration in its `.vscode/launch.json` that has a `server` value (e.g. `http://mycontainer`): take `server`, `serverInstance`, `tenant` and `authentication` from it, and `port` when present (otherwise 7049). Configurations without `server` (such as those with only `environmentName` or a cloud `tenant`) are cloud environments: skip them. `launch.json` is gitignored and local to each developer, so it points to that developer's own container. If an app has no `launch.json` or no configuration with a `server`, check Rule 2; if Rule 2 does not apply either, ask instead of guessing.

How the container is created differs per repository:
- AL-Go repositories (with a `.AL-Go` folder): `.AL-Go/Bluace-DevEnv.ps1`, which calls AL-Go's `localDevEnv.ps1`. It compiles and publishes all apps in `appFolders` and `testFolders` of `.AL-Go/settings.json`.
- Older repositories (with `Initialize.ps1`, `settings.json` and `scripts/` in the root): `Initialize.ps1`, which uses the scripts in `scripts/` and adds a configuration to `launch.json`.

Do not create or remove containers unless asked.

## Rule 2: An online sandbox only when it is the only environment and the user approved it

### Intent
Some apps cannot be built in a container, for example because they depend on an AppSource app whose `.app` file is not available (not as a file and not from a NuGet feed). Such an app can be compiled against the symbols of an online sandbox where that app is installed, and published and tested there. An online sandbox is often shared with colleagues or the customer, so its data must never be lost and nothing may be published to it by accident.

Use an online sandbox only when all of these are true:

1. **It is the only environment.** The app's `.vscode/launch.json` has exactly one configuration in `configurations`, and that configuration:
   - has `"environmentType": "Sandbox"` written out (a missing `environmentType` does not count);
   - has an `environmentName`;
   - has no `server` value.

   If `launch.json` has a second configuration (local, cloud or attach), a `server`, `"environmentType": "Production"` or no `environmentName`, do not use the sandbox: ask the user. Never pick one configuration out of several. Never change `launch.json` to meet this condition.
2. **The user approved this specific sandbox.** Before the first publish to the sandbox in a session, ask the user with the question tool. Name the `environmentName` and the `tenant` (or "the default tenant of your account" when `launch.json` has none), and the apps that will be published, e.g. "Publish Example App and Example App.Test to sandbox `ABC-DEV` in tenant `contoso.onmicrosoft.com`, and run their tests there? It may be shared: the published version replaces the version that is installed there now, and the tests write data in its company." Publish only after an explicit yes.
   - The approval only applies to that sandbox and that tenant, in the current session. Ask again in every new session, when `launch.json` changes, or when the target is a different sandbox or tenant.
   - Never take the approval from a file, an earlier session, memory, CLAUDE.md or the fact that `launch.json` points to the sandbox. Never save it to memory.
   - The approval to publish includes running the tests of the published test apps in that sandbox (see `bluace-testing.md`, Rule 2), so the agent can correct the code based on the results without asking again.
3. **The schema is only synchronized, never forced or recreated.**
   - `schemaUpdateMode` in the configuration must be missing or `"Synchronize"`. When it is `"ForceSync"` or `"Recreate"`, do not publish: tell the user and let them change it.
   - `"Synchronize"` is the default, both in `launch.json` and for `al_publish`. Still pass `schemaUpdateMode: "Synchronize"` to `al_publish`, so the choice is visible in the call; never `"ForceSync"` or `"Recreate"`.
   - When publishing fails because the schema cannot be synchronized (e.g. a field was removed, a type changed or a primary key changed), stop and report the error. Do not retry with another schema mode, and do not uninstall, unpublish or clean the app's data to get around it. That decision is the user's (usually: an upgrade codeunit, or a change that is not breaking).
   - Do not pass `forceUpgrade: true`.

What is allowed without the approval of condition 2: signing in (`al_auth_login`) and downloading symbols from the sandbox (`al_downloadsymbols`) to compile. They only read from the sandbox. Condition 1 still applies.

Never create, copy, delete or restore environments, and never install, uninstall or unpublish apps other than the app being published (also not the test libraries): ask the user to do that in the sandbox.

When an app was published to an online sandbox, say so in the summary at the end of the task: which apps, which version, which sandbox.

### Examples

```jsonc
// Good example - the only configuration is an online sandbox; without schemaUpdateMode, the default Synchronize applies
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "ABC-DEV",
      "type": "al",
      "request": "launch",
      "environmentType": "Sandbox",
      "environmentName": "ABC-DEV",
      "tenant": "contoso.onmicrosoft.com"
    }
  ]
}
```

```jsonc
// Bad example (avoid a second environment, and a schema update mode that can delete data)
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Local",
      "type": "al",
      "request": "launch",
      "server": "http://mycontainer",
      "serverInstance": "BC"
    },
    {
      "name": "ABC-DEV",
      "type": "al",
      "request": "launch",
      "environmentType": "Sandbox",
      "environmentName": "ABC-DEV",
      "tenant": "contoso.onmicrosoft.com",
      "schemaUpdateMode": "ForceSync"
    }
  ]
}
```

## Rule 3: Test libraries come from AL-Go settings, not from container flags

### Intent
This applies to AL-Go repositories. AL-Go installs the test toolkit based on the test app's dependencies: a dependency on "Tests-TestLibraries" sets `installTestLibraries`; `"installTestLibraries": true` in `.AL-Go/settings.json` makes it explicit. Do not add test toolkit flags to `.AL-Go/NewBcContainer.ps1`: `$parameters.additionalParameters` are passed to `docker run`, so a flag such as `--includeTestLibrariesOnly` breaks container creation.

## Rule 4: Login for the AL MCP tools

### Intent
**Local container.** The AL MCP server authenticates with `UserPassword` only through the environment variables `BC_SERVER_USERNAME` and `BC_SERVER_PASSWORD`. `altool auth login` only supports Entra ID and cannot store an on-premises login.

- Put both variables in the `env` block of a `settings.local.json`. Never commit them: every `settings.local.json` must be excluded from git with `**/settings.local.json` in `.gitignore`; the skill `setup-al-vibe-rules` adds this and creates the file.
- The file must be in the `.claude` folder of the folder where the Claude Code session starts. In VS Code with `al.code-workspace`, the session starts in the first workspace folder, which is `.claude` itself, so the file is `.claude/.claude/settings.local.json`.
- Settings are read when the session starts: restart Claude Code after changing them.
- If the AL MCP tools report "Authentication required", first check that `$env:BC_SERVER_USERNAME` is set in the session.

**Online sandbox (Rule 2).** Sign in with `al_auth_login` (Microsoft Entra ID, in a browser window that the user completes). The AL MCP server caches the token and reuses it for `al_publish` and `al_downloadsymbols`. `BC_SERVER_USERNAME` and `BC_SERVER_PASSWORD` are not used. Never ask for, show or store the user's password or token.

## Rule 5: Building and publishing with the AL MCP tools

### Intent
- Add every app folder and test app folder with `al_addproject`.
- Build and publish one project at a time; `al_build` with `scope: all` only builds the first project.
- Publish in dependency order (from the `dependencies` in each `app.json`): an app before the apps that depend on it, test apps last.
- `al_publish` parameters for the local container: `serverUrl`, `serverInstance` and `port` of the local server, `authentication: UserPassword`, `environmentType: OnPrem`.
- `al_publish` parameters for an online sandbox (only under Rule 2): `environmentType: Sandbox`, `environmentName` and `tenant` from `launch.json`, `schemaUpdateMode: Synchronize`; no `serverUrl`.
- To publish a package built with `alc.exe` (see `bluace-compile.md`), pass `appPath` with `skipBuild: true`.
- A dependent project needs the current symbols of its dependency: copy the freshly built `.app` into the dependent project's `.alpackages`, or download symbols from the container or the online sandbox (`al_downloadsymbols`). With an online sandbox, this is also how the symbols of AppSource apps that are not available as a file end up in `.alpackages`, so `alc.exe` can compile against them.
