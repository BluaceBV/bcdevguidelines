# Bluace: Procedure Order

In which order triggers, global variables, procedures and events are placed in an object.

## Rule 1: Triggers, global variables, procedures by usage, events

### Intent
An object is read from top to bottom in the order in which its code runs. After its properties and its layout, fields, keys or actions, it has four sections, always in this order:

1. **Triggers**, in their natural order of execution.
2. **Global variables**: the object's `var` section.
3. **Procedures**, ordered by usage.
4. **Events**: event publishers (`[IntegrationEvent]`, `[BusinessEvent]`, `[InternalEvent]`).

An object without triggers starts with its global variables; an object without global variables goes from the triggers straight to the procedures.

Field triggers (`OnValidate`, `OnLookup`, ...) and action triggers stay with their field or action; this rule is about the object's own triggers, procedures and events.

**Triggers.** Never place a procedure above or between the triggers. Order the triggers as the platform runs them, not alphabetically (the standard application often sorts them alphabetically, a leftover from the C/AL export format; do not copy that):
- Codeunit: `OnRun`.
- Table and table extension: `OnInsert`, `OnModify`, `OnDelete`, `OnRename` (each `OnBefore...` variant before and `OnAfter...` variant after its trigger).
- Page and page extension: `OnInit`, `OnOpenPage`, `OnFindRecord`, `OnNextRecord`, `OnAfterGetRecord`, `OnAfterGetCurrRecord`, `OnNewRecord`, `OnInsertRecord`, `OnModifyRecord`, `OnDeleteRecord`, `OnPageBackgroundTaskCompleted`, `OnPageBackgroundTaskError`, `OnQueryClosePage`, `OnClosePage`.
- Report: `OnInitReport`, `OnPreReport`, `OnPostReport`.
- Upgrade and install codeunits: the order of the platform (`OnCheckPreconditions...`, `OnUpgrade...`, `OnValidateUpgrade...`; per company before per database).

**Procedures by usage.** A reader follows the code from caller to callee, so a procedure is followed by the procedures it calls:
- Place a called procedure directly below the procedure that calls it, and its own callees directly below it (depth first). Procedures called from triggers follow the order of those triggers.
- A procedure called from several places goes below its first caller, or below the last of a group of callers that are placed next to each other (e.g. two drill-down procedures that both open the same list).
- Do not sort by name or by access modifier (`local`, `internal`, public): the call order decides.
- Procedures that call nothing in the object and are not called within it (such as the procedures of a table that only pass on to a helper codeunit) keep a logical order, for example the order of the actions they serve.

**Events.** All event publishers come after the last procedure. Among themselves they may follow the order of the procedures that raise them.

### Examples

```al
// Good example - triggers, procedures by usage, events
page 90151 "Payment Link Act. ABC"
{
    // layout, actions ...

    trigger OnOpenPage()
    begin
        Rec.InsertIfNotExists();
    end;

    trigger OnAfterGetCurrRecord()
    begin
        CalculateCueFieldValues();
    end;

    trigger OnPageBackgroundTaskCompleted(TaskId: Integer; Results: Dictionary of [Text, Text])
    begin
        if TaskId = TaskIdCalculateCue then
            Rec.FillFromDictionary(Results);
    end;

    var
        TaskIdCalculateCue: Integer;

    local procedure CalculateCueFieldValues()
    begin
        // ...
    end;
}

codeunit 90152 "Payment Link Helper ABC"
{
    var
        LinkExpiredErr: Label 'Payment link %1 has expired.', Comment = '%1 = Payment link id';

    internal procedure IsLinkExpired(PaymentLink: Record "Payment Link ABC"): Boolean
    begin
        exit(CurrentDateTime() - PaymentLink."Created At" >= GetExpiryInterval());
    end;

    local procedure GetExpiryInterval() Interval: Duration
    begin
        Interval := 24 * 60 * 60 * 1000;
        OnGetExpiryInterval(Interval);
    end;

    internal procedure OpenInvoiceTomorrow()
    begin
        // ...
        ShowInvoices(InvoiceNos);
    end;

    internal procedure OpenInvoiceAfterTomorrow()
    begin
        // ...
        ShowInvoices(InvoiceNos);
    end;

    local procedure ShowInvoices(InvoiceNos: List of [Code[20]])
    begin
        // ...
    end;

    [IntegrationEvent(false, false)]
    local procedure OnGetExpiryInterval(var Interval: Duration)
    begin
    end;
}
```

```al
// Bad example (avoid global variables above the triggers, procedures between triggers, alphabetical triggers, grouping by access modifier and events between procedures)
page 90151 "Payment Link Act. ABC"
{
    var
        TaskIdCalculateCue: Integer;

    trigger OnAfterGetCurrRecord() ...
    local procedure CalculateCueFieldValues() ...
    trigger OnOpenPage() ...
    trigger OnPageBackgroundTaskCompleted(...) ...
}

codeunit 90152 "Payment Link Helper ABC"
{
    internal procedure IsLinkExpired(...)
    internal procedure OpenInvoiceTomorrow()
    internal procedure OpenInvoiceAfterTomorrow()

    local procedure GetExpiryInterval() ...
    [IntegrationEvent(false, false)]
    local procedure OnGetExpiryInterval(...)
    local procedure ShowInvoices(...)
}
```
