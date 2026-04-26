---
name: jwtRefreshProvider onboarding gap
description: jwtRefreshProvider lives in CalcHome, not active during onboarding wizard — revisit if onboarding adds Supabase calls
type: project
---

`jwtRefreshProvider` is activated in `apps/gst_calculator/lib/views/calc_home.dart:250` via `ref.watch(jwtRefreshProvider)`. It is NOT active during the CalcOnboardingWizard flow.

**Why:** Decided 2026-03-12 during lazy Supabase init work. CalcHome is the root shell for all 4 tabs and always mounted post-onboarding. The provider guards with `if (!SupabaseRestClient.isInitialized) return` so it's a safe no-op before Supabase exists.

**How to apply:** If the GST Calculator onboarding wizard (`apps/gst_calculator/lib/onboarding/`) ever adds Supabase-dependent steps, hoist `ref.watch(jwtRefreshProvider)` to `GstCalculatorApp` (the MaterialApp-level ConsumerWidget). Compare with the "Hoist to GstCalculatorApp" pattern: wrap MaterialApp in a ConsumerWidget that watches jwtRefreshProvider from app launch.
