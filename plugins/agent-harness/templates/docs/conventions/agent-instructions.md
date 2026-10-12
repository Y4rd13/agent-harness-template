# Agent Instruction Files

Goal: every instruction an agent reads has one home, and it changes only for a reason. An instruction
file is read at the start of every session, so each line costs attention on every task, and a stale or
contradictory line makes the agent pick one rule arbitrarily. This guide says what goes where, how to
write it, and when a change is justified.

## What lives where

| File | Holds | Loaded |
| --- | --- | --- |
| `AGENTS.md` | Facts true in every session: commands an agent cannot guess, the stack, architecture rules, Critical Invariants, boundaries, and the merge rules. Canonical for every tool | Every session, by every agent that reads `AGENTS.md` |
| `CLAUDE.md` | `@AGENTS.md` plus notes that apply to Claude Code alone (skills, plugins, MCP) | Every Claude Code session |
| `CLAUDE.local.md` | Personal notes for one machine, never committed | Every Claude Code session, for you only |
| `.claude/rules/*.md` | A rule that matters for one part of the tree, with a `paths:` glob | Claude Code only, when it reads or edits a matching file |
| `.claude/skills/<name>/SKILL.md` | A procedure of several steps run on request (open a pull request, review one) | Its description always; its body when invoked |
| `.claude/settings.json` | Permissions and hooks: what is allowed, asked or denied, enforced by the client | Every Claude Code session |
| `docs/conventions/*.md` | The long form of a convention: rationale, examples, edge cases | When an agent or a person opens it |
| `README.md` | The workflow table: every CI workflow, its trigger and what it checks | When opened |
| CI checks | Every rule a machine can verify | On every pull request |
| `.claude/claims.json` | Statements in the instruction files bound to a fact a check verifies | On every pull request, if you run the claims check |

`CLAUDE.md` keeps its `@AGENTS.md` line. Recent Claude Code versions read `AGENTS.md` on their own only
when no `CLAUDE.md` exists, and some sessions cannot read it directly; the import works in all of them and
never loads the file twice. An import does not save context: the imported file loads at launch.

## Where a new instruction goes

Take the first answer that applies.

1. A machine can check it: write the check (a CI step, a lint rule, a hook). The prose line, if any,
   names the check instead of restating the rule. A security or data invariant still keeps its one line
   under Critical Invariants (step 5), so every agent and reviewer sees it.
2. It must hold every time, with no exception: a check or a hook, never prose alone. An instruction is
   advisory; a hook or a required check is enforcement.
3. It is a procedure of several steps that an agent follows on request: a skill, not a section here.
4. It matters only for files under one path: a `.claude/rules/` file with `paths:`.
5. It is a security or data invariant every agent and reviewer must see: `AGENTS.md`, Critical
   Invariants. Not `.claude/rules/`, because only Claude Code reads that directory.
6. It is a fact true in every session: `AGENTS.md`, in the section it belongs to.
7. It needs more than two lines of explanation: `docs/conventions/<topic>.md`, with a one-line pointer
   under "Detail on demand".

## What never goes in an instruction file

- Information that changes often. A list of every CI workflow changes with every workflow change; it
  lives in the `README.md` table, and `AGENTS.md` keeps only the merge rules.
- Anything an agent can read from the code: directory listings, file-by-file descriptions, dependency
  versions it can read from the manifest.
- Standard conventions the model already knows, and self-evident advice ("write clean code").
- Long explanations and tutorials: they go to `docs/conventions/`.

## How to write a line

- Per line, ask "Would removing this cause an agent to make a mistake?" If not, cut it.
- Make it verifiable: a path, a command, a number or a named constraint, so a reviewer can tell when it
  was broken. "Run the formatter before pushing", not "keep the code tidy".
- One `IMPORTANT` per file at most. When many lines are emphasised, none stands out.
- Maintainer notes go in block-level HTML comments: Claude Code strips them before the file reaches the
  model.
- No two lines may contradict each other across `AGENTS.md`, `CLAUDE.md`, `.claude/rules/` and
  `docs/conventions/`. When two disagree, the agent picks one and does not say so.

## Size

A check fails a pull request when `AGENTS.md` passes 200 lines, `CLAUDE.md` passes 40, a rule passes 200,
or a rule lacks `paths:`. A rule whose `paths:` is missing or does not parse loads in every session with
no error. 200 lines is the target the Claude Code documentation gives for adherence; other agents cap or
truncate long instruction files too, so a file inside 200 lines fits every tool.

## When a change is justified

A change to `AGENTS.md`, `CLAUDE.md` or a rule needs one of these reasons, named in the pull request body:

1. An agent got the same convention wrong twice, or a review caught something an agent should have known.
2. A new invariant or an exception to one, shipped with the code that needs it.
3. A statement in the file is false today. Fix it; if a check could have caught it, add the check or a
   claim.
4. The same correction was typed into a session a second time. A procedure pasted a third time becomes a
   skill instead.
5. A model upgrade made a workaround unnecessary: delete the workaround.

Not reasons: a new CI workflow (update the `README.md` table), a renamed script that only the CI section
mentioned, or wording that says the same thing differently.

## Review and maintenance

- Put `AGENTS.md`, `CLAUDE.md` and `.claude/**` in `CODEOWNERS`, so a change needs an owner's review. Keep
  a feature change and an instruction change in separate pull requests when the instruction change is not
  needed to review the feature.
- Run an audit of the instruction files once a month and after every model upgrade: stale references,
  contradictions, files over their limit, workarounds an older model needed.
- After a change, check that the agent's behavior moved. A line that changes nothing is a line to cut.

## Sources

- Claude Code memory and `CLAUDE.md`: https://code.claude.com/docs/en/memory
- Claude Code best practices: https://code.claude.com/docs/en/best-practices
- Claude Code features overview: https://code.claude.com/docs/en/features-overview
- The `AGENTS.md` format: https://agents.md
