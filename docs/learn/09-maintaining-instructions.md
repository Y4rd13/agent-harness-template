# 09. Maintaining the instructions

Instruction files rot like code, but nothing fails when they do: the agent just follows a stale line. Treat
them like code: reviewed, tested, pruned.

## When to change them

A change to `AGENTS.md`, `CLAUDE.md` or a rule needs a reason, named in the pull request body:

1. An agent got the same convention wrong twice, or a review caught something an agent should have known.
2. A new invariant, or an exception to one, shipped with the code that needs it.
3. A statement in the file is false today.
4. The same correction was typed into a session a second time. A procedure pasted a third time becomes a
   skill.
5. A model upgrade made a workaround unnecessary.

Not reasons: a new CI workflow (update the README table instead), or rewording that says the same thing.

The `add-rule` skill walks the ladder of
[agent-instructions.md](../../plugins/agent-harness/templates/docs/conventions/agent-instructions.md) for
you: check, hook, skill, path-scoped rule, Critical Invariants, `AGENTS.md`, or a conventions document.

## How to write a line

- Verifiable: a path, a command, a number. "Run the formatter before pushing", not "keep it tidy".
- One `IMPORTANT` per file at most.
- No contradictions across files: when two lines disagree, the agent picks one and does not say which.
- Maintainer notes in HTML comments, which never reach the model.

## Pruning

Per line: "Would removing this make the agent make a mistake?" If not, delete it. Candidates:

- workarounds for an older model;
- anything the agent can read from the code;
- lines that restate a CI check (name the check instead);
- long explanations (move them to `docs/conventions/` and leave one line).

## The monthly check

Once a month, and after every model upgrade:

1. Run `/agent-harness:audit`: limits, rules that load everywhere, stale paths, missing commands,
   contradictions.
2. Open `/memory` and `/context` in a fresh session and confirm what actually loads.
3. Try a small task that touches a path-scoped rule and watch whether the agent follows it.
4. Delete what no longer changes behavior.

## Claims keep lines honest

When a line states a fact a machine can check ("the CI gate is called X", "this rule exists"), add a claim
to `.claude/claims.json`. The claims check fails when the fact or the quoted line changes, so the file is
updated in the same pull request that made it false.

## Sources

- Claude Code best practices: https://code.claude.com/docs/en/best-practices
- Claude Code docs, memory: https://code.claude.com/docs/en/memory
- Certification guide 3.5, iterative refinement:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-5-iterative-refinement
