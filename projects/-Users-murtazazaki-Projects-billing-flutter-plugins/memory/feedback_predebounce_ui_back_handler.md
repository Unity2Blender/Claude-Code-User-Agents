---
name: UI back handlers must not rely on debounced state alone
description: System-back / pop handlers in surfaces with debounced search/input must check raw controller state too, otherwise a user typing-then-pressing-back before debounce settles can pop the route instead of clearing the input.
type: feedback
originSessionId: 2c5fcb68-16a2-4df2-b8c9-85d16216119a
---
P1 DEFAULT — When a UI surface has a debounced search/input notifier, the system-back handler must consult raw controller state in addition to (or instead of) the debounced derivative.

**Why:** 2026-05-02 plan rejection. `apps/gst_calculator/lib/views/videos_tab/videos_home_view.dart:316-321` `_handleSystemBack` only checks `searchNormalizedQueryProvider.isNotEmpty` (the debounced 350ms derivative). If the user types a single character and presses back within 350ms (common on Android), the debounced provider is still empty, `PopScope.canPop` evaluates `!inSearchMode` as true, the route pops, and the user loses context they expected to clear. Users perceive this as a buggy back button.

**How to apply:**
- In any back-handler that gates `PopScope.canPop`:
  1. Check raw `_textController.text.isNotEmpty` (or equivalent raw state) FIRST.
  2. If raw is non-empty, clear it (and the debounced provider) and consume the back event.
  3. Only fall through to system pop if both raw and debounced are clear.
- Pair this with `canPop: !(inSearchMode || _textController.text.isNotEmpty)` so PopScope itself blocks the pop until clear.
- This is a permutation in `state_permutation_testing` rules — write a widget test that types a character and presses system back BEFORE the debounce settles (FakeAsync 100ms), asserting the search input clears and the route does NOT pop.
- The same pattern applies to wizard step inputs, chat inputs, party search sheets, and any debounced filter UI.
