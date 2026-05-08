---
name: Dart privacy is library-scoped — `_Foo` widgets cannot live in separate files
description: A class named `_VideosBottomActionBar` cannot be imported from `_videos_bottom_action_bar.dart` into `videos_home_view.dart`. Dart privacy is library-scoped, not class-scoped. Keep the widget in the same library (same file or part), or make the class public.
type: feedback
originSessionId: 5fbffcbe-4672-4cdb-a9ca-c76034c28dfa
---
Dart privacy rule: a leading-underscore identifier (`_Foo`, `_bar()`, `_baz`) is visible only within the same Dart **library**. By default every `.dart` file is its own library, so `_Foo` declared in `a.dart` is invisible to `b.dart` even with an explicit `import`.

**Three valid patterns:**
1. **Same file:** declare the widget in the same file that uses it. Smallest, most common.
2. **`part` / `part of`:** split a single library across multiple files. Bookkeeping cost; only worth it for large modules.
3. **Public class with `@visibleForTesting` / convention:** export under a public name; restrict via docs and reviewers.

**Anti-pattern (broken):** create `_videos_bottom_action_bar.dart` containing `class _VideosBottomActionBar extends StatelessWidget {...}`, import it from `videos_home_view.dart`, try to construct `_VideosBottomActionBar(...)`. Compiles to "the name `_VideosBottomActionBar` isn't defined."

**Why:** Round-4 audit on `videos-tab-majestic-harbor` flagged exactly this in a planned implementation step. Lock #17 corrected: `VideosBottomActionBar` is public and lives in `videos_home_view.dart` (smallest correct change; extract only when a 2nd consumer exists).

**How to apply:** Whenever a plan suggests extracting a private widget into a separate file "for clarity," check whether the consumer is in the same library. If not, choose explicitly between (1) keep in same file, (2) use `part`, or (3) make the class public.
