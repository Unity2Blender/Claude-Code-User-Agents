- Once a plan has been implemented or Features implemented, Git commit & push with a brief message of changes (Like a Hook on Stop or SessionEnd of planning and implementation)
- When a change adds or alters a significant module/feature in any directory, create or update a CLAUDE.md in that directory or sub-directory.
- When a directory is tagged/referred in a message, during Read/Explore/Plan phases, check if that directory or any relevant sub-directory contains a CLAUDE.md file. Read and refer to these CLAUDE.md files to gain context about the present working directory (pwd) or tagged directories before proceeding with the task.
- Always prefer WebSearch to fetch the latest official documentation, specs, RFCs, or changelogs related to the technology, framework, library, or API being discussed, rather than relying on outdated or cached knowledge. 
- When interviewing me or presenting decisions via AskUserQuestion, follow this protocol:

  **Scope:** Apply ranked trade-off format to architecture, implementation, and design decisions only. Trivial questions (yes/no, simple confirmations, naming) stay simple.

  **Preamble:** Before each AskUserQuestion call, write 1-2 sentences explaining WHY you recommend the top option and what ranking criteria you applied.

  **Option Ranking — always rank options best-first using these priority weights:**
  1. **Correctness & reliability** — financial accuracy, data integrity, GST compliance. Wrong calculations are catastrophic.
  2. **Bundle size & performance** — lighter, faster options rank higher. Target: minimal client footprint.
  3. Context-adaptive third priority (read `.claude/.session-context` if it exists):
     - `database` sessions → data integrity, query performance
     - `ui-implementation` sessions → UX simplicity, fewer moving parts
     - `ui-research` sessions → design system alignment, accessibility
     - `business` sessions → speed-to-market, competitive advantage
     - `general` / unknown → simplicity & maintainability

  **Option Format — every option description MUST use this compact formula:**
  > `Best for [specific case]. [1-line why it suits]. Trade-off: [1-line downside].`

  **Label Convention:** First option label ends with `(Recommended)`. Other options are ordered by descending fit.

  **Example:**
  ```
  Preamble: "I recommend Supabase RPC because server-side GST calculation guarantees correctness and saves ~1.5MB client bundle."

  Option 1 label: "Supabase RPC (Recommended)"
  Option 1 desc:  "Best for your billing case: server-side calc is always correct. Cuts client bundle by ~1.5MB. Trade-off: requires internet, adds ~50ms latency."

  Option 2 label: "Client-side calculation"
  Option 2 desc:  "Best for offline-first apps. Fast, no network needed. Trade-off: ~1.5MB bundle increase, risk of client/server calc drift."
  ```

  **Interview behavior:** Ask in-depth, non-obvious questions about technical implementation, UI & UX, concerns, and tradeoffs. Continue interviewing until complete, then write the spec to file.