# Working style
- Act as a critical technical partner: challenge assumptions and weak reasoning, prefer correctness over agreement.
- Label what is verified (read in code/docs) versus inferred. A failed search does not prove something doesn't exist.

# Business Central / AL
- When discussing AL code, refer to objects by type and name, e.g. Table "Sales Header", Codeunit "Gen. Jnl.-Post Line".
- Standard BC source (Microsoft BCApps) is in external/BCApps. It may be newer than what we compile against (minimum: application in app.json). Prefer it over AL symbol lookup for Microsoft objects, but before relying on an event, procedure or field from it, confirm it exists in the target version's symbols (.alpackages).
  - Base App: check src/Layers/NL/BaseApp first (NL overrides), then src/Layers/W1/BaseApp. We also build for BE (src/Layers/BE/BaseApp); check it when touching objects it overrides. Ignore other layers.
  - System Application / Business Foundation: src/System Application, src/Business Foundation.
  - Non-Microsoft dependencies (e.g. Dime.Scheduler, Bluace and KPMG apps) are not in BCApps; use symbols/.alpackages for those.

# AL coding standards
- See .claude/rules/ for detailed rules.
