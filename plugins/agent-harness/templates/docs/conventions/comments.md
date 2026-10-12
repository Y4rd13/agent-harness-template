# Comments and Docstrings

Goal: less noise. Comments are objective, direct and minimal. Explain WHY, not WHAT. When the code and
its types already say it, write nothing. This guide applies to every language in the repository,
configuration and CI files included.

## Principles

- **WHY, not WHAT.** The reader knows the language. A comment earns its place by explaining intent, a
  non-obvious constraint, or a decision the code cannot state by itself.
- **Prefer no comment over a redundant one.** A comment that restates the code rots independently of it.
- **Names first.** Clear names and complete types remove most of the need for prose. Rename before you
  annotate.
- **No changelog in the code.** Comments describe the code as it is now. Ticket keys, pull request
  numbers, dates, measurements and "added/changed/fixed" narration belong in the commit, the pull request
  and the ticket.
- **Delete dead code.** Git remembers it.

## When to write what

| Situation | Write |
| --- | --- |
| Name and types are self-explanatory | Nothing |
| Public function with a side effect, a non-obvious behavior or a caller contract | One-line docstring |
| A line does something surprising | One comment on that line, saying why |
| Parameter or return semantics that types cannot convey | A structured docstring (rare) |

## Noise to avoid

```text
# Avoid: restates the code
i = i + 1  # increment i

# Prefer: nothing, or the reason
i = i + 1  # skip the header row

# Avoid: history in the code
"""Rotate the password (settings page flow, TICKET-123)."""

# Prefer: what it does now
"""Rotate the password and revoke existing sessions."""
```

## Mannered prose

Mannered prose replaces a direct statement with metaphor or flourish: "a dial worth turning" instead of
"a parameter worth varying". It makes the reader work harder and adds connotations the writer did not
choose. Say what you mean with the literal word. The short instruction for an agent is: "Remove all
mannered prose." It applies to comments, commit messages and pull request bodies.

## Enforcement

The checkable parts (no ticket keys and no em-dashes inside comments) can run in CI; see the
`comment-conventions` check in the plugin's check reference.
