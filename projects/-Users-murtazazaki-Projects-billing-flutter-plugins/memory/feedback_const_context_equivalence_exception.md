# Const-Context Equivalence Exception

Date: 2026-04-29

## Pattern

When a `static const` initializer blocks a theme-token reference, a literal is acceptable only if it is verified equivalent to the intended token at HEAD and the code documents the migration trigger.

## Conditions

- The preferred token cannot compile in the const context.
- The literal exactly matches the token for all current variants.
- A nearby comment states the equivalence, the reason for the exception, and the trigger for refactoring.
- Tests pin the token equivalence or behavior that depends on it.

## Example

MBBook Modern's item-table header paints `theme.pdfPrimaryColor`, but its cell config is a `static const TableCellConfig`. `theme.pdfOnPrimaryColor` cannot be referenced there, and every current `InvoiceThemes.all` theme maps `pdfOnPrimaryColor` to `PdfColors.white`. The smallest correct fix is therefore `headerColor: PdfColors.white` with a comment saying to refactor to a per-render factory if `pdfOnPrimaryColor` ever stops being white.

## Boundary

This is an exception, not a style preference. Use theme tokens everywhere token access is available.
