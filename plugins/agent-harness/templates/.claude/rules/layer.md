---
paths:
  - "src/<layer>/**"
---

# <Layer name> (src/<layer>)

<!--
One rule file per part of the tree that has its own conventions: the HTTP layer, the data layer, the UI
components. Name what an agent would get wrong in this directory, with the file that shows the pattern.
If a line here is a security or data invariant, put it in AGENTS.md under Critical Invariants instead:
only Claude reads this file.
-->

- <What this layer may and may not do>, for example: handlers never query the database directly; they call a service.
- <Required decorator, wrapper or middleware on new entry points>, with the pattern in `<path/to/example>`.
- <Where user-facing text or API documentation goes in this layer.>
- <The check that enforces any of the above, if one exists>: `<check name>`.
