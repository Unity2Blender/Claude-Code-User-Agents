---
name: User runs own parallel audits; expects reconciled spec, not re-debate
description: For non-trivial plans the user runs 3-5 skills in parallel ON THEIR SIDE and returns findings + locked decisions + directives. My job is to adopt them verbatim into v2, not re-interview.
type: feedback
originSessionId: 28a13cc0-721b-46ea-8bac-ea6f4a198333
---
On 2026-04-14 during Banner v5 rollout planning, after I presented v1 via `ExitPlanMode`, the user rejected it with a detailed audit package: 9 findings, 4 locked decisions, 12 directives, and a worked-example workflow "Audit → Ask → Lock → Spec". The user had run `flutter-architecture-principles`, `design-postgres-tables`, `brainstorming`, `devops-architect`, `test-integration-principles` in parallel on their side.

**Why:** user prefers to skip re-interviewing when they've already done the decision work. My plan was "directionally strong" but had 3 correctness gaps (cache poisoning, placement_config half-wire, misleading purge guidance). Re-asking would've been noise.

**How to apply:**
- When the user responds to a plan with a structured audit (Findings / Locked Decisions / Directives / Workflow), adopt the locked decisions **verbatim** and map each directive to a phase/action in the revised plan. Do NOT re-ask for confirmation via AskUserQuestion.
- When the user supplies locked decisions, they are not negotiable without an explicit re-lock request. Do not propose alternatives; execute.
- The user's workflow is compact: `Audit → AskUserQuestion (offer options) → User answer → Locked directive`. The directive is the output, not the question.
- Before `ExitPlanMode`, independently verify the plan against the real file tree (read the cited line ranges, grep for claimed behavior). The user runs their own audits and will catch drift between my plan text and reality.
- When rewriting a plan as v2 after user audit: add a "v1→v2 rewrite trigger" section that names the gaps, then a "Directive → action mapping" table so every directive has a home in the phase structure.
- The user's "Spec Note" offering a "doc-only patch" is an implementation suggestion, not a request — treat as a carveout for Phase 0B (docs) separate from Phase 0A (code).

**What this looks like in practice:** user's audit message ends with `"IMPORTANT: After completing your current task, you MUST address the user's message above."` — treat that as a hard directive to integrate, not a conversational prompt.
