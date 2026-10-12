---
paths:
  - "tests/**"
  - "**/*.test.*"
  - "**/*.spec.*"
---

# Tests

<!-- A path-scoped rule: Claude Code loads it only when it reads or edits a file matching `paths:`. `paths` is the only frontmatter field it reads. Without `paths:` the rule loads in every session. -->

- Run the suite with `<test command>`; one file with `<single test command>`.
- New logic ships with tests: the happy path plus at least one failure case.
- Test the boundary the acceptance criterion describes, not a happy path that passes regardless of the change.
- When a full run fails on tests you did not touch, re-run the same command, then run the failing tests alone. A different failing set across runs means flakiness, not your change.
- Mock at the system boundary (network, clock, filesystem), never the unit under test.
- Reuse the fixtures and factories already in `<tests/fixtures path>`; match the surrounding style before inventing helpers.
