# Bluace: Comments

How comments are used in AL code. This supplements Rule 3 (Code Documentation and Comments) in `al-code-style.md`.

## Rule 1: Comments only where the code cannot speak for itself

### Intent
Code explains itself through clear names and small procedures. Do not add comments that describe what the code does or why a design was chosen; that belongs in the specification, the work item or the commit message.

Comments are kept in two cases only:
- XML documentation comments (`///`) on procedures that other objects call, describing how to call them: what the procedure does for the caller, its parameters and its return value (see `al-code-style.md`, Rule 3).
- A justification for a construct that is not self-explanatory and is only correct for a reason the code does not show. Always:
  - `Commit()`: why the transaction must be committed at that point;
  - an intentionally empty statement, such as `if Rec.Get() then;`: why the result is ignored and what happens when the read fails.

A justification explains why, not what. Put it on the line before the statement or as a trailing comment on the same line. These are also the cases where the analyzers ask for a comment (DocumentationCop DC0001 for `Commit()`, DC0003 for empty statements).

### Examples

```al
// Good example - documentation of how to call, and justifications only
/// <summary>
/// Creates a payment link for the posted sales invoice.
/// </summary>
/// <param name="InvoiceNo">The number of the posted sales invoice.</param>
/// <returns>The id of the new payment link.</returns>
internal procedure CreateLinkForInvoice(InvoiceNo: Code[20]): Text[50]
begin
    // ...
end;

local procedure SaveStatus(var PaymentLink: Record "Payment Link ABC")
begin
    PaymentLink.Modify(true);
    // The status must be kept when the payment provider call that follows fails and rolls back.
    Commit();
end;

local procedure RereadLink(var PaymentLink: Record "Payment Link ABC")
begin
    if PaymentLink.Get(PaymentLink.Id) then; // Keep the shown values when the link was deleted meanwhile, instead of raising an error.
end;
```

```al
// Bad example (avoid comments that repeat the code or explain the design)
local procedure SaveStatus(var PaymentLink: Record "Payment Link ABC")
begin
    // Save the payment link
    PaymentLink.Modify(true);
    Commit();
end;

local procedure GetRefreshInterval() Interval: Duration
begin
    Interval := 15 * 60 * 1000; // 15 minutes
end;
```
