# PartySearchNotifier Permission Microtask

Any Riverpod-built test that resolves `PartySearchNotifier` directly or
transitively through `partySearchProvider` must install the permission_handler
MethodChannel mock before pumping or reading the provider.

## Why

`PartySearchNotifier.build()` schedules a microtask that calls
`Permission.contacts.status`. Without a mock for
`MethodChannel('flutter.baseflow.com/permissions/methods')`, tests throw
`MissingPluginException`. This was confirmed in micro-slice `ec170bb5` across
four unrelated test files.

## How

Call `installPermissionHandlerMock()` in `setUp()` and
`uninstallPermissionHandlerMock()` in `tearDown()`. For widget tests using
`tester.usePortraitViewport()`, install the permission mock after the viewport
extension.

Channel contract:

- Channel: `flutter.baseflow.com/permissions/methods`
- `checkPermissionStatus` returns `int` (`0` denied, `1` granted)
- `requestPermissions` returns `Map<int, int>` (`Permission.value` to status)
- `Permission.contacts.value == 2`
