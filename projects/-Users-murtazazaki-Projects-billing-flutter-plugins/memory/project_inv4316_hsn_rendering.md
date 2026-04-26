---
name: INV4316 HSN Column Not Rendering
description: Production bug where HSN auto-hides due to broken _tryGetProperty reflection; data-wins philosophy adopted for mandatory columns
type: project
originSessionId: 67fde43b-b9c0-4864-aa0a-9dd0f19e7b45
---
Production bug discovered 2026-04-09: HSN column missing from all 7 column-aware templates despite data existing in line items.

**Root cause**: `ColumnBuilder._tryGetProperty()` calls `(item as dynamic).toJson()` but `ItemSaleInfo` has no `toJson()` method. Silent `NoSuchMethodError` → null → auto-hide triggers. Latent: Gate 7 GST override also broken for typed inputs.

**Fix**: `_getColumnValue()` delegates to `ColumnValueExtractor.getValue()` when `item is ItemSaleInfo`. New `dataForcedWhenPresent` flag on `ColumnConfig` forces HSN visible when data exists, overriding preset and user hiddenColumns.

**Why:** Data integrity > user preference for compliance-critical columns. HSN is informational (always safe to show). GST % stays user-controllable (financial, can confuse on non-GST bills).

**How to apply:** When adding new columns with `autoHideIfEmpty: true`, always verify the extraction path works for typed `ItemSaleInfo` objects, not just Maps. Consider whether `dataForcedWhenPresent: true` is appropriate.
