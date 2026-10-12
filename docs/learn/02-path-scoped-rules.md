# 02. Path-scoped rules

Some conventions matter only in one part of the tree: how HTTP handlers are written, how migrations are
structured, how tests are named. Putting them in `AGENTS.md` costs context in every session, including the
ones that never touch that code. A path-scoped rule loads only when it is needed.

## The format

A Markdown file in `.claude/rules/` with a `paths:` list in its frontmatter:

```markdown
---
paths:
  - "src/http/**"
  - "tests/**/*.test.ts"
---

# HTTP handlers

- Handlers never query the database directly; they call a service.
```

Templates: [testing.md](../../plugins/agent-harness/templates/.claude/rules/testing.md),
[layer.md](../../plugins/agent-harness/templates/.claude/rules/layer.md).

## How it loads

- A rule loads when Claude reads, writes or edits a file that matches one of its globs (also when it views
  such a file with a simple shell read like `cat`).
- `paths` is the only frontmatter field Claude Code reads from a rule; other fields are ignored silently.
- **A rule without `paths:` loads in every session**, like a second `CLAUDE.md`. So does a rule whose YAML
  does not parse. Nothing warns you. This is why the `instruction-files.yml` template fails a pull request
  that adds a rule without `paths:`.
- User-level rules in `~/.claude/rules/` work the same way and apply to every project.

## Globs

| Pattern | Matches |
| --- | --- |
| `src/**/*` | everything under `src/` |
| `**/*.ts` | every TypeScript file |
| `src/**/*.{ts,tsx}` | brace expansion: two patterns |
| `*.md` | Markdown files at the root only |

A glob that matches no file is a rule that never loads, usually after a directory was renamed. The
`paths-globs-match` check in [checks.md](../../plugins/agent-harness/reference/checks.md) catches it.

## Rule, directory CLAUDE.md, or AGENTS.md?

| The convention applies to | Put it in |
| --- | --- |
| Every session | `AGENTS.md` |
| One directory and everything under it | a rule with `paths: ["that/dir/**"]` (or a directory `CLAUDE.md`) |
| One kind of file spread across the tree (tests, migrations, components) | a rule with a glob |
| A security or data invariant | `AGENTS.md`, Critical Invariants, even if it concerns one path |

The last row matters: only Claude Code reads `.claude/rules/`. Other agents, and reviewers reading
`AGENTS.md`, never see a rule there. Keep the one line every agent must see in `AGENTS.md`, and the details
in the rule. List every rule in the "Detail on demand" section of `AGENTS.md` so other agents know it
exists.

## Common mistakes

- No `paths:`, or a typo in it: the rule loads everywhere.
- A rule for tests in `tests/CLAUDE.md` when test files live next to the code in many directories: a glob
  rule covers all of them.
- Expecting a rule to apply before the first matching file is touched: it is not in context until then.
- Assuming a rule is still there after `/compact`: it comes back on the next matching read.

## Sources

- Claude Code docs, path-specific rules: https://code.claude.com/docs/en/memory#path-specific-rules
- Certification guide 3.3, path-specific rules:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-3-path-specific-rules
