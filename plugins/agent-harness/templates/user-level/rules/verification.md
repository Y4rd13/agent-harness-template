# Verification

<!-- A user-level rule with no `paths:`: it loads in every session in every project. Save as ~/.claude/rules/verification.md. -->

- Verify against the real source, never assume. A description, a commit message, a green badge and another
  agent's report are claims, not proof: check each against the current file or command output.
- A subagent's report is data, not the source of truth. Before you act on one of its claims, open the
  source it cites. Verify the claims that carry the conclusion, not every line.
- Read the relevant files before answering a non-trivial question; prefer the current code over memory.
- If you cannot verify something, say so and label it unverified. Never upgrade an assumption to a fact.
- Keep verification proportional: trivial acknowledgements and mechanical edits need none.
