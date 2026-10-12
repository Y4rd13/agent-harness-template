---
name: audit
description: Audit a repository's agent harness (AGENTS.md, CLAUDE.md, .claude/rules, claims, CI workflows) for broken limits, rules that load everywhere, missing imports, stale paths, unpinned actions, workflow injection and contradictions. Read-only. Use when the user asks to audit, check or review their CLAUDE.md, AGENTS.md, rules or instruction files, or after a model upgrade.
when_to_use: 'Trigger phrases: audit my harness, check my CLAUDE.md, revisa mis instrucciones, are my rules loading, why is the agent ignoring AGENTS.md.'
allowed-tools: Read, Grep, Glob, Bash(git ls-files *), Bash(git rev-parse *)
---

# Audit the agent harness

Run every check in `${CLAUDE_PLUGIN_ROOT}/reference/checks.md` against the repository in the current
directory, by reading files. Change nothing: this skill only reports.

1. Read `${CLAUDE_PLUGIN_ROOT}/reference/checks.md` in full. Each `## <check-id>` section is one check.
2. List the repository's files with `git ls-files` (fall back to Glob outside a git repository). Note
   which of these exist: `AGENTS.md`, `CLAUDE.md`, `.claude/rules/`, `.claude/claims.json`,
   `.github/workflows/`, `package.json`, `Makefile`, `justfile`, `pyproject.toml`.
3. Run each check in the order of the reference. Skip a check only when its inputs do not exist, and say
   so in the report. Count lines with the Read tool's line numbers, not by estimate.
4. For `contradictions`, compare the actual statements; report one only when the two lines cannot both be
   followed.
5. Report one table, errors first:

   | file:line | check | severity | fix |
   | --- | --- | --- | --- |

   Then one line per skipped check with its reason, and a last line:
   `errors: <n>, warnings: <n>, checks run: <n> of 12`.

Every row must point at a line you read. Do not report style preferences, and do not propose rewriting a
file that passes. If the repository has no harness at all, say so and suggest `/agent-harness:init`.
