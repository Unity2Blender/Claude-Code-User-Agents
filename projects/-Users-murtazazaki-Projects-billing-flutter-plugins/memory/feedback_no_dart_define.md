---
name: No --dart-define for feature flags
description: User strongly prefers flutter_dotenv over --dart-define for all app configuration including microservice URLs and feature flags
type: feedback
---

Never use `--dart-define` / `String.fromEnvironment` for microservice URLs or feature flags. Always use `flutter_dotenv` via `flutter.env`.

**Why:** User considers `--dart-define` unnecessary complexity ("dart-define nonsense"). The project already uses `flutter_dotenv` for all other configs (SupabaseConfig, OcrConfig, PaymentGatewayConfig). Compile-time defines also fail silently in CI when Codemagic doesn't inject them.

**How to apply:** When adding new microservice config (URLs, feature flags, timeouts), create a singleton `*Config` class following the `OcrConfig` pattern — synchronous `initialize()` reading from `dotenv.env` (already loaded by `SupabaseConfig.initialize()`). Add LOCAL/PROD env vars to `flutter.env`.
