# Bluace: Automated Tests

Supplements `al-testing.md` with what applies to the test apps in this repo.

## Rule 1: Running tests with the AL MCP tools

### Intent
- Run tests with `al_run_tests` per test codeunit, with `projectPath` (the test app folder, so its `launch.json` is used). Without it the runner gets a 401. Do not pass `company`: the tests run in the default company of the local environment.
- The AL MCP test runner cannot run tests that use TestPages. Put TestPage scenarios in a separate UI test codeunit; they run in the AL-Go pipeline.
- Publish the app and the test app first (see `bluace-dev-environment.md`).

## Rule 2: Tests in an online sandbox

### Intent
Tests run in an online sandbox when the user approved publishing to that sandbox in the session (`bluace-dev-environment.md`, Rule 2). That approval includes running the tests, without a separate question, so the agent can correct the code based on the results. The sandbox may be shared, and Microsoft limits testing there ([Testing the application](https://learn.microsoft.com/dynamics365/business-central/dev-itpro/developer/devenv-testing-application#environment-testing-support-and-limitations)): no large numbers of tests, no test method longer than 15 minutes, and preferably not more than one or two hours of tests a day.
- Run only the test codeunits of the feature you changed, not the whole test app.
- Tests that `Commit()` (see Rule 5) can leave data in the sandbox's company. Name them in the approval question of `bluace-dev-environment.md`, Rule 2, so the user knows this before approving.
- The test framework and test libraries the test app depends on must already be installed in the sandbox. If they are missing, tell the user; do not install them.

## Rule 3: No external communication in tests

### Intent
Tests never call external services, in containers and in online sandboxes alike.
- Replace HTTP calls with a mock that implements the System Application Interface "Http Client Handler". Inject it through an `[InternalEvent]` in the app's API client, and expose internals to the test app with `internalsVisibleTo`.
- As backstops, point the base URL at an unresolvable `.invalid` host and use a dummy token.
- Do not use `TestHttpRequestPolicy`; it is on-premises only.

## Rule 4: Permission tests must start Permissions Mock

### Intent
"Library - Lower Permissions" does nothing unless Codeunit "Permissions Mock" is started, and the AL MCP test runner does not start it. A permission test would then pass without any restriction.
- Start it in the test (`if not PermissionsMock.IsStarted() then PermissionsMock.Start();`, e.g. through a test library procedure) and stop it again afterwards.
- Call "Library - Lower Permissions" (e.g. `SetO365BusFull()`, `AddPermissionSet(...)`) directly in the test procedure, not in a helper (CodeCop AA0087).
- Create test data before lowering permissions.

## Rule 5: asserterror rolls back uncommitted data

### Intent
`asserterror` rolls back every uncommitted change of the test, including test data created before it. Commit test data (with a comment explaining why) that the test checks or needs after an `asserterror`.

## Rule 6: Every test starts with its scenario and acceptance criterion

### Intent
The first line of every test procedure is a `[SCENARIO]` comment with the number of the acceptance criterion it covers (from the feature's `acceptance.md`) and a one-sentence description, followed by the `[GIVEN]`/`[WHEN]`/`[THEN]` comments. Prefix the test procedure name with the same number. When one test covers several criteria, list them all; when several tests cover one criterion, each repeats that number.

A `[GIVEN]`, `[WHEN]` or `[THEN]` comment that directly follows code gets an empty line before it, so each section stands out. No empty line before such a comment when it follows `begin` or another comment (e.g. `[SCENARIO]`, or a `[GIVEN]` without code of its own). The same applies to these comments in helper procedures of the test codeunit.

### Examples

```al
// Good example
[Test]
procedure B7_ActiveLinkBlocksNewLink()
var
    // ...
begin
    // [SCENARIO B7] An active link is refreshed first and then blocks a new link.
    // [GIVEN] INV-A with an active link that is still active at the payment provider
    LibraryPaymentLink.CreateActiveLink(PaymentLinkABC);

    // [WHEN] A link is created
    asserterror PaymentLinkABC.CreateLinkForInvoice(InvoiceNo);

    // [THEN] The link is refreshed first, then an error names the active link; no POST
    Assert.ExpectedError(StrSubstNo(ActiveLinkExistsErr, PaymentLinkABC.Id));
end;
```

```al
// Bad example (avoid a section comment directly after code, and an empty line after begin)
begin

    // [SCENARIO B7] An active link is refreshed first and then blocks a new link.
    // [GIVEN] INV-A with an active link that is still active at the payment provider
    LibraryPaymentLink.CreateActiveLink(PaymentLinkABC);
    // [WHEN] A link is created
    asserterror PaymentLinkABC.CreateLinkForInvoice(InvoiceNo);
    // [THEN] The link is refreshed first, then an error names the active link; no POST
    Assert.ExpectedError(StrSubstNo(ActiveLinkExistsErr, PaymentLinkABC.Id));
end;
```

## Rule 7: Test app dependencies in BC 29

### Intent
In BC 29, "Tests-TestLibraries" does not pass on all libraries to the test app. Add these as direct dependencies when used: "Library Variable Storage" (`5095f467-0a01-4b99-99d1-9ff1237d286f`) and "Permissions Mock" (`40860557-a18d-42ad-aecb-22b7dd80dc80`), next to "Library Assert", "Any" and "Tests-TestLibraries".

## Rule 8: Tests must pass in every client language

### Intent
The app is translated (en-US, nl-NL), so error texts depend on the language of the user who runs the tests. Never assert on an English fragment of an app message.
- Declare the expected message in the test codeunit as a translatable label with the exact same English text as the app's label (same `Comment`), and assert the full message: `Assert.ExpectedError(StrSubstNo(PaidLinkCannotBeArchivedErr, LinkId))`.
- The test app has a dedicated XLIFF file for every language the app supports, named like the app's files: `Translations/<test app>.<language>.xlf` (e.g. `Example App.Test.en-US.xlf`, `Example App.Test.nl-NL.xlf`). When the app gets a new language, add the matching file to the test app too.
- Every label in the test app gets a target in each of these files, copied from the app's translation of the same source text in that language.
- Build captions in the message with `FieldCaption(...)`, not literals.
- Fragments that are language-independent (HTTP status codes, URLs, texts returned by the mock) may stay literal.
