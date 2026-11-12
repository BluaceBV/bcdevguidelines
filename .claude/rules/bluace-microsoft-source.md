# Bluace: Microsoft Objects and Source Code

Where to look up Microsoft objects (Base Application, System Application, Business Foundation and Microsoft's other first-party apps): their fields, events, procedures and source code. There is no local copy of the Microsoft source (such as an `external/BCApps` folder); everything comes from the MCP servers of the bc-observatory plugin.

## Rule 1: bc-observatory for facts, bc-code-atlas for code, Learn for documentation

### Intent
The bc-observatory plugin (marketplace `https://waldo1001.github.io/waldo.BCObservatory/marketplace.json`) connects three MCP servers. Each answers a different question:

| Question | Server | Typical tools |
|---|---|---|
| Which object, id, fields, keys, events, public procedures; obsolete state; in which versions and countries; what changed between versions; who calls it | `bc-observatory` | `bc_search`, `bc_get_object`, `bc_cat`, `bc_diff_object`, `bc_localization` |
| What a procedure actually does: its body, its exact signature, what it calls, code found by meaning | `bc-code-atlas` | `bcatlas_resolve_node`, `bcatlas_get_signature`, `bcatlas_get_procedure_body`, `bcatlas_get_neighbors`, `bcatlas_search` |
| Documentation of AL methods, properties and functional features; AL code samples | `microsoft-learn` | `microsoft_docs_fetch` on the url of a Learn hit, `microsoft_code_sample_search` |

- These three servers come only through the plugin. Do not register them in a project's `.mcp.json` or as a user MCP server, also not Microsoft Learn (`https://learn.microsoft.com/api/mcp`): a second registration shows the same tools twice. When a claude.ai connector for the same server is connected (e.g. Microsoft Learn), its tools may appear instead of the plugin's; use them the same way.
- Start with the observatory to identify the object, then use the atlas for behaviour. Use `bcatlas_resolve_node` with the object type, never a free-text `bcatlas_get_node`. The plugin's `bcobs-grounding` skill describes this flow.
- Refer to objects by type and name, e.g. Table "Sales Header", Codeunit "Gen. Jnl.-Post Line". Never take object ids, field numbers, event signatures or versions from memory: look them up.
- If the bc-observatory tools are not available (also not after ToolSearch), say so and point to the plugin, which `setup-al-vibe-rules` enables for the project. Do not guess Microsoft code instead.
- The atlas is a free server run by one person: no sweeps over many objects, no loops of calls. If it does not answer, say so and continue with the observatory's facts.

### Examples

```
// Good example - identify with the observatory, verify the behaviour in the atlas
bc_get_object(type: "table", idOrName: "Sales Header")                      -> fields, events, procedures, versions
bcatlas_resolve_node(object_type: "table", object_name: "Sales Header", member: "InitInsert")
bcatlas_get_procedure_body(label: "<id from resolve_node>")                 -> what InitInsert really does
```

```
// Bad example (avoid)
Read external/BCApps/src/Layers/W1/BaseApp/Sales/Document/SalesHeader.Table.al   // no local Microsoft source anymore
bcatlas_get_node(label: "Sales Header")                                          // fuzzy, may return a page control
```

## Rule 2: Check the version the app is built against

### Intent
The servers do not hold the version the app compiles against by default:
- the observatory covers the AL objects of BC28, BC29 and BC30 (vNext) and the changes from BC23; `bc_get_object` says which versions an object exists in, `bc_diff_object` what changed;
- the atlas queries W1 BC28 (`w1-28`) by default. `bcatlas_list_warm_versions` lists the other builds that can be queried right away; pass their `commit_sha` as `version` with the `country`.

The target version is `application` in the app's `app.json` (see `bluace-project-context.md`).
- State the version of every code fact from the atlas, e.g. "w1-28".
- When the target version differs from the version you looked at, check with `bc_get_object` or `bc_diff_object` that the object, field, event or procedure exists in the target version and did not change in a way that matters.
- Before relying on an event, procedure or field, confirm that it exists in the app's symbols in `.alpackages` (e.g. with `al_symbolsearch`), as the compiler sees them.
- Do not call `bcatlas_request_version` unless the user asks for it: it starts a build on the maintainer's server that takes several minutes.

## Rule 3: Country layers

### Intent
Bluace builds for the Dutch localization (NL), and some apps also for Belgium (BE). The country layers change W1 objects and add their own.
- For an object of the Base Application, check what the NL layer adds or changes first (`bc_get_object` with `country: "NL"`, or `bc_localization`), then W1. Check BE as well when the app is built for BE and the object is changed by the BE layer. Ignore other countries.
- The atlas answers for W1 unless a warm build of the country is passed (`bcatlas_list_warm_versions(country: "nl")`). When there is none, say that the code shown is W1 and name what the observatory reports as changed in the country layer.

## Rule 4: Non-Microsoft dependencies

### Intent
The observatory and the atlas only contain Microsoft's apps. For other dependencies (e.g. Dime.Scheduler, Bluace and partner apps), use the symbols in `.alpackages` (e.g. with `al_symbolsearch`) or the source repository of that app.
