# Bluace: Folder Structure

Objects are grouped by feature under `src/`, with two extra folders: `General` for everything that depends directly on another app, and `Reports` for reports and report extensions. This extends Rule 2 (Feature-Based Folder Organization) in `al-code-style.md`.

Rules 1–4 apply to a new app, and to an existing app whose folders are restructured completely, on purpose and in one go. When functionality is added to an existing app, Rule 5 applies: one app has one folder style.

## Rule 1: Feature folders without number prefixes

### Intent
Folder names under `src/` describe the feature in PascalCase and have no number prefix. Numbered folders (`01-setup`, `99-permissionsets`) are an old way of working and are not used in new apps.

- One folder per feature, containing that feature's own objects: tables, pages, enums, and the helper codeunits of its own tables (e.g. `Setup/`, `PaymentLink/`).
- `PermissionSets/` for permission sets.
- File names follow `<ObjectNameWithoutSuffix>.<ObjectType>.al` (see `al-naming-conventions.md`).

## Rule 2: General folder for everything that depends on another app

### Intent
`src/General/` holds all functionality that depends directly on another app (the Base Application or other dependencies):
- table extensions, page extensions and enum extensions (report extensions go in `Reports/`, see rule 3);
- the helper codeunits of those table extensions;
- all subscriber codeunits (`<publisher> Subscr. <suffix>`).

This keeps a clear border between the app's own features and the places where it hooks into other apps.

### Examples

```
// Good example
Example App/src/
├── General/
│   ├── EnvTriggerSubscr.Codeunit.al
│   ├── PaymentMethod.TableExt.al
│   ├── PaymentMethodHelper.Codeunit.al
│   ├── PaymentMethods.PageExt.al
│   ├── PostedSalesInvoice.PageExt.al
│   ├── SalesInvHeaderHelper.Codeunit.al
│   └── SalesInvoiceHeader.TableExt.al
├── PaymentLink/
│   ├── APIClient.Codeunit.al
│   ├── EnvironmentType.Enum.al
│   ├── PaymentLink.Table.al
│   ├── PaymentLinkHelper.Codeunit.al
│   └── PaymentLinks.Page.al
├── PermissionSets/
│   ├── ExampleApp.PermissionSet.al
│   └── ExampleAppAdmin.PermissionSet.al
└── Setup/
    ├── Setup.Page.al
    ├── Setup.Table.al
    └── SetupHelper.Codeunit.al
```

```
// Bad example (avoid number prefixes and extensions mixed into feature folders)
src/
├── 01-setup/
│   ├── PaymentMethod.TableExt.al
│   └── Setup.Table.al
└── 99-permissionsets/
```

## Rule 3: Reports folder for reports and report extensions

### Intent
`src/Reports/` holds reports and report extensions, together with their helper codeunits and their layout files. A report extension gets a helper codeunit like a table extension does: the report extension only calls it, and the helper holds the logic.

Naming:
- A report extension is named after the base report, without the base app's suffix and with the app's own suffix: "Sales - Invoice ABC" extends "Sales - Invoice RBLC". Its helper codeunit has the same base name: "Sales - Invoice Helper ABC".
- A layout file has exactly the same name as its report or report extension file, with the layout extension, so both sort next to each other: `SalesInvoice.ReportExt.al` + `SalesInvoice.ReportExt.rdlc`, `MyReport.Report.al` + `MyReport.Report.rdlc`.

### Examples

```
// Good example
Example App/src/
└── Reports/
    ├── SalesInvoice.ReportExt.al
    ├── SalesInvoice.ReportExt.rdlc
    └── SalesInvoiceHelper.Codeunit.al
```

## Rule 4: Test apps mirror the feature folders

### Intent
Test apps use the same feature folder names (`Setup/`, `PaymentLink/`, ...) for their test codeunits, plus `Common/` for shared libraries and mocks. Tests for objects in `General/` go in the feature folder whose behaviour they test.

## Rule 5: Existing apps keep their folder style

### Intent
When functionality is added to an existing app, keep its folders as they are and follow the naming style the app already uses, even when that style differs from Rules 1–4. Do not move or rename existing folders as part of a feature. Changing the folder style is a separate, deliberate decision that is applied to the whole app at once (and then Rules 1–4 apply).

- Determine the style from the existing folders under `src/`: number prefixes or not, upper or lower case, separators.
- A new feature folder gets the next free number and the same casing and separators when the app uses numbered folders.
- Extensions, subscribers, reports and permission sets go in the existing folder where the app keeps them (e.g. its general or reports folder), not in a new `General/` or `Reports/` folder.

### Examples

```
// Good example - existing app with numbered, lowercase folders
src/
├── 01-general/
├── 02-migration/
├── 03-reports/
├── 04-paymentlink/      <- new feature, next free number, same style
└── 99-permissionset/
```

```
// Bad example (avoid two folder styles in one app)
src/
├── 01-general/
├── 02-migration/
├── 03-reports/
├── PaymentLink/         <- new feature in another style
└── 99-permissionset/
```
