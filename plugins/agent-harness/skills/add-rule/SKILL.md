---
name: add-rule
description: Decide where a new instruction for coding agents belongs (a CI check, a hook, a skill, a path-scoped rule, Critical Invariants, AGENTS.md or a conventions doc) and propose the exact change. Use when the user says "from now on", "de ahora en adelante", "always do X", "never do X", "add a rule", "remember that in this repo", or corrects the same thing a second time. Prefix the instruction with dry-run to only print the proposal.
when_to_use: 'Trigger phrases: from now on, de ahora en adelante, always do, never do, add a rule, agrega una regla, remember that in this repo, the same correction repeated.'
argument-hint: "[dry-run:] <instruction>"
allowed-tools: Read, Grep, Glob, Bash(git ls-files *)
---

# Put a new instruction in the right place

The instruction is in the arguments, or in the user's last message. If it starts with `dry-run`, print
the proposal and write nothing.

1. Read `${CLAUDE_PLUGIN_ROOT}/templates/docs/conventions/agent-instructions.md` (the ladder and its
   reasons), then the repository's `AGENTS.md`, `CLAUDE.md` and `.claude/rules/`. If the repository has
   no harness yet, say so and suggest `/agent-harness:init`.
2. Walk the ladder in order and stop at the first step that applies:
   1. A machine can check it: propose the check (a lint rule, a CI step, or a check from
      `${CLAUDE_PLUGIN_ROOT}/reference/checks.md`), plus at most one line that names it.
   2. It must hold every time, with no exception: a hook in `.claude/settings.json` or a required CI
      check. Prose alone is advisory.
   3. It is a procedure of several steps run on request: a skill in `.claude/skills/<name>/SKILL.md`.
   4. It matters only for files under one path: a `.claude/rules/<topic>.md` with a `paths:` glob that
      matches at least one tracked file.
   5. It is a security or data invariant every agent and reviewer must see: one line under
      `## Critical Invariants` in `AGENTS.md`.
   6. It is a fact true in every session: one line in the matching section of `AGENTS.md`.
   7. It needs more than two lines of explanation: `docs/conventions/<topic>.md` plus a one-line pointer
      under "Detail on demand".
3. Before proposing, check the result:
   - it does not contradict an existing line in `AGENTS.md`, `CLAUDE.md`, `.claude/rules/` or
     `docs/conventions/` (if it does, propose replacing that line, not adding a second one);
   - the target file stays within its limit (200 lines for `AGENTS.md` and rules, 40 for `CLAUDE.md`);
   - the line is verifiable: a path, a command, a number or a named constraint;
   - a new rule is listed in the "Detail on demand" section of `AGENTS.md`.
4. Report: the step you chose and one sentence on why the earlier steps did not apply, then the diff.
   When step 1 or 2 applies, also say what the user loses if they settle for prose instead.
5. Unless this is a dry run, ask for approval and write only what was approved.
6. Remind the user that a change to an instruction file needs a reason in the pull request body (see
   "When a change is justified" in the guide).
