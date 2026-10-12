---
name: init
description: Set up an agent harness in the current repository from the agent-harness templates - AGENTS.md, a CLAUDE.md import shim, path-scoped rules, .claude/settings.json with a safety hook, a ship-pr skill, a reviewer subagent, PR template, CODEOWNERS and CI workflows - filled with the repository's real commands. Shows every file before writing and writes only after approval. Pass dry-run to only print the proposal.
disable-model-invocation: true
argument-hint: "[dry-run]"
allowed-tools: Read, Grep, Glob, Bash(git ls-files *), Bash(git status *), Bash(git branch *), Bash(git check-ignore *), Bash(git config --get *)
---

# Set up the agent harness

The templates are in `${CLAUDE_PLUGIN_ROOT}/templates/`; the rules for what goes where are in
`${CLAUDE_PLUGIN_ROOT}/templates/docs/conventions/agent-instructions.md`. Read that guide first. Work on
the repository in the current directory.

**Dry run.** If the arguments contain `dry-run`, or the AskUserQuestion tool is not available (a
non-interactive `claude -p` run), do steps 1 to 4 only: print the proposal and write nothing.

## 1. Read the repository

Read before asking anything. Collect:

- the files (`git ls-files`), the languages, the package manager and its lockfile;
- the real commands: the `scripts` of `package.json`, the targets of a `Makefile` or `justfile`, the tasks
  in `pyproject.toml`; note which exist for install, dev, test, a single test, lint, format, type check and
  build. Never invent a command that is not defined;
- the source and test directories, and the layers inside the source tree (for example `src/http`,
  `src/core`); the existing CI under `.github/workflows/`; the default branch (`git branch`);
- existing harness files: `AGENTS.md`, `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules/`,
  `.claude/settings.json`, `.github/pull_request_template.md`, `CODEOWNERS`;
- whether git ignores `.claude/`: `git check-ignore -v .claude/settings.json`. Many global gitignores do,
  and then the harness files never reach the repository.

## 2. Ask only what the files cannot tell

One AskUserQuestion call, at most four questions, recommended option first:

1. Which pieces to install (multi-select): instruction files (recommended), Claude Code config
   (recommended), deterministic CI (recommended), AI review layer (not preselected: it needs a model API
   secret and a mode variable, see the comments at the top of `ai-review.yml`).
2. Which areas need the user's OK before an agent changes them (multi-select, from what you found: auth,
   payments, migrations, infrastructure, public API).
3. The Critical Invariants: offer the ones the code suggests (for example "every query filters by the
   owner" if you saw owner-scoped queries), and let the user add their own through "Other".
4. Only if it is ambiguous: which directories deserve their own path-scoped rule.

## 3. Build the proposal

For each piece chosen, map a template to a target path and fill it:

- `AGENTS.md`: the real commands, the real stack, the layers you found, the user's invariants and
  ask-first areas, the CI checks that will exist. Remove every `<...>` placeholder: fill it with a fact
  from the repository or delete the line, and list each deleted line in the report so the user can add
  it. Keep the HTML comment notes out of the generated file. Stay under 200 lines.
- `CLAUDE.md`: the shim, `@AGENTS.md` on its own line, under 40 lines.
- `.claude/rules/`: `testing.md` with `paths:` matching the real test files, and one rule per layer that
  has its own conventions, each with a `paths:` glob that matches at least one tracked file. Add each rule
  to the "Detail on demand" list in `AGENTS.md`.
- `.claude/settings.json`: the allow list from the real commands; keep the push and `.env` rules and the
  hook unchanged.
- `.claude/skills/ship-pr/SKILL.md`, `.claude/agents/reviewer.md`, `.github/pull_request_template.md`,
  `.github/CODEOWNERS` (ask for the owner team if unknown, or leave the file out), `.github/dependabot.yml`
  with the real ecosystem.
- `docs/conventions/agent-instructions.md` and `docs/conventions/comments.md`: copy them whenever the
  generated `AGENTS.md` points to them, so no pointer leads to a missing file.
- CI: `ci.yml` with one step per command that works today and the toolchain setup the stack needs (pinned
  by commit SHA), `pr-title.yml`, `secret-scan.yml`, `instruction-files.yml`; and, only if chosen,
  `ai-review.yml` with `.github/prompts/ai-review.prompt.md` and `.github/ai-review.schema.json`. A command
  whose tool, configuration or lockfile is missing (for example `npm ci` without a lockfile, `tsc` without
  a `tsconfig.json`) stays out of `ci.yml`: a pull request that is red from its first run teaches the team
  to ignore CI. List each one in the report with what it needs.
- `.gitignore`: add `CLAUDE.local.md` and `.claude/settings.local.json`. If step 1 found `.claude/` ignored,
  add `!.claude/` and say why.

Existing files are never overwritten. When a target exists, propose a diff that keeps what is there. When
there is a `CLAUDE.md` and no `AGENTS.md`, propose moving the shared conventions into `AGENTS.md` and
leaving `CLAUDE.md` as the shim plus its Claude-only lines.

## 4. Show it

Print one table, then the full proposed `AGENTS.md`:

| path | action | from template |
| --- | --- | --- |
| `AGENTS.md` | create / diff | `templates/AGENTS.md` |

Below it, list the placeholder lines you deleted and any command you could not find. In a dry run, stop
here and say that nothing was written.

## 5. Write, after approval

Ask once: write all, choose files, or cancel. Write only what was approved, with the Write and Edit tools.

## 6. Check what you wrote

Run the checks of `${CLAUDE_PLUGIN_ROOT}/reference/checks.md` on the result, as the `audit` skill does, and
fix any error in the files you just wrote. Then tell the user the next steps: review the diff, commit on a
feature branch, and for the AI review layer, add the secret and set `AI_REVIEW_MODE=shadow` first.
