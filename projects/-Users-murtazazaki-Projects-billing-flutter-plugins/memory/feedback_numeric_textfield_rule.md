---
name: Numeric TextField formatting rule
description: Don't use .toString() or .toStringAsFixed(2) for TextField controller sync from numeric state — use formatNumericClean() + _isUserEditing guard
type: feedback
---

Don't use `.toString()` or `.toStringAsFixed(2)` for TextField controller sync from numeric state. Use `formatNumericClean()` from `lib/core/utils/numeric_format.dart`. Add `_isUserEditing*` flag to skip sync during user typing. Only sync on external state changes.

**Why:** `.toString()` converts 3.0 → "3.0", causing cursor displacement. `.toStringAsFixed(2)` always shows "120.00" even when user typed "120". Both patterns cause cursor jumping when `didUpdateWidget` or `ref.listen` syncs state back to controller while user is typing.

**How to apply:** When a numeric TextFormField uses a TextEditingController that syncs from external state (via `didUpdateWidget` or `ref.listen`), always: (1) format with `formatNumericClean()`, (2) guard sync with `_isUserEditing*` flag set in `onChanged`, (3) reset flag in the sync guard's user-editing branch. Pattern proven in `definition_basic_step.dart` and `stock_entry_body.dart`.
