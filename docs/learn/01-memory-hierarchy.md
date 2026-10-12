# 01. The memory hierarchy: CLAUDE.md, AGENTS.md and friends

An AI coding assistant starts every session knowing nothing about your repository. Instruction files are
how it learns your commands, your architecture and your rules before it touches anything. This lesson
covers which files Claude Code reads, in what order, and how `AGENTS.md` fits in.

## The levels

| Level | Location | Shared with | Use it for |
| --- | --- | --- | --- |
| Managed policy | `/etc/claude-code/CLAUDE.md` (Linux), set by IT | Everyone on the machine | Organization rules nobody can switch off |
| User | `~/.claude/CLAUDE.md` and `~/.claude/rules/` | Only you, every project | How you like to work |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md`, plus `.claude/rules/` | The team, through git | The repository's conventions |
| Local | `./CLAUDE.local.md` (gitignored) | Only you, this project | Your sandbox URLs, your test data |
| Directory | `src/payments/CLAUDE.md` | The team | Rules for one subtree, loaded on demand |

Templates: [AGENTS.md](../../plugins/agent-harness/templates/AGENTS.md),
[CLAUDE.md](../../plugins/agent-harness/templates/CLAUDE.md),
[CLAUDE.local.md.example](../../plugins/agent-harness/templates/CLAUDE.local.md.example),
[user-level/](../../plugins/agent-harness/templates/user-level/CLAUDE.md).

## How they load

- **Concatenated, not overridden.** Every file found is added to the context. Nothing "wins": if two lines
  contradict each other, the model may follow either one. Across directories, content is ordered from the
  filesystem root down to your working directory.
- **Ancestors at launch, subdirectories on demand.** Files in the working directory and above it load when
  the session starts. A `CLAUDE.md` in a subdirectory loads once Claude reads or edits a file there.
- **It is context, not configuration.** The model reads these files and usually follows them, but nothing
  forces it to. What must always hold goes in settings, hooks or CI (lesson 04 and 06).
- **Compaction.** After `/compact`, the project-root `CLAUDE.md` is re-read from disk. Nested files and
  path-scoped rules come back only when a matching file is read again. An instruction you typed only in the
  chat is gone: if it matters, put it in a file.

## Imports

A line made of `@` and a path imports that file:

```markdown
@AGENTS.md
@docs/conventions/testing.md
```

Relative paths resolve from the importing file. Imports nest up to four hops. An import does not save
context: the imported file loads at launch, the same as if you had pasted it.

## AGENTS.md: one file for every agent

`AGENTS.md` is a plain-Markdown convention read by many coding agents. Keep your conventions there, once,
and make `CLAUDE.md` a shim:

```markdown
@AGENTS.md

## Claude-specific notes
- Skills, plugins or MCP servers that only Claude Code uses.
```

Recent Claude Code versions read `AGENTS.md` on their own, but only when there is no `CLAUDE.md` or
`CLAUDE.local.md` in the working directory or above it, and some sessions cannot read it directly. The
`@AGENTS.md` import works in every session and never loads the file twice, so keep it.

## Size and style

- Aim for under 200 lines per file. Longer files cost context in every session and are followed less.
- Per line, ask: would removing this make the agent make a mistake? If not, cut it.
- Block-level HTML comments (`<!-- note -->`) are stripped before the model sees the file. Use them for
  notes to human maintainers; the templates do.
- Do not paste what the agent can read from the code (directory listings, dependency versions) or what it
  already knows ("write clean code").

## Seeing what loaded

- `/memory` lists the memory files of the session and opens one for editing.
- `/context` shows what is in the context window, with a "Memory files" section.

Neither command loads anything.

## Common mistakes

- Team conventions in `~/.claude/CLAUDE.md`: they work on your machine only, and a new teammate's agent
  behaves differently.
- A `CLAUDE.local.md` added to a repository that relies on Claude reading `AGENTS.md` directly: it counts
  as a `CLAUDE.md` file, so `AGENTS.md` stops loading. Use the import.
- A `CLAUDE.md` that says "read AGENTS.md" in words instead of importing it: Claude sees the file only if it
  decides to open it.
- A directory `CLAUDE.md` for a convention that spans directories (for example tests everywhere): use a
  path-scoped rule (lesson 02).

## Exam vs current behavior

The certification guide accepts either `./CLAUDE.md` or `./.claude/CLAUDE.md` as the project file; both
are valid. It notes that there is no `@import` keyword: the import is a line that starts with `@`.

## Sources

- Claude Code docs, memory: https://code.claude.com/docs/en/memory
- The `AGENTS.md` format: https://agents.md
- Certification guide 3.1, CLAUDE.md hierarchy:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-1-claude-md-hierarchy
