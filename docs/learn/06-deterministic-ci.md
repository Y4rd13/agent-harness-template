# 06. The deterministic CI layer

An agent can read a rule and still break it. CI is where a rule stops being advice: a check that fails the
pull request does not depend on anyone having read anything. The first question for any new convention is
"can a machine check this?" If yes, write the check and let the instruction file name it.

## Principles

- **One gate job, every failure at once.** Run lint, format, type check, tests and build as steps of one
  job, each with `if: ${{ !cancelled() }}`, so a red run lists every failure and the author fixes them in
  one push. One job also installs dependencies once.
- **CI runs the same commands as humans and agents.** The commands in `AGENTS.md` are the commands in
  `ci.yml`. If they differ, the agent's local check passes and CI fails.
- **A check that cannot fail is not a check.** A job with `continue-on-error: true`, or an `if:` that skips
  it on the paths that matter, reports green whatever happens.
- **Never weaken a failing check to get green.** Fix the cause, or raise it with a maintainer. Say so in
  `AGENTS.md` under Never, because agents under pressure will try.
- **The harness tests the harness.** Every checker script you write gets its own test, and the workflows
  run when `.github/` or `.claude/` change.

## The checks

| Check | Why | Ships as |
| --- | --- | --- |
| Lint, format, type check, test, build | The basics, in one job | [ci.yml](../../plugins/agent-harness/templates/.github/workflows/ci.yml) |
| Conventional Commit title | The squash title decides the release | [pr-title.yml](../../plugins/agent-harness/templates/.github/workflows/pr-title.yml) |
| Secret scan | Git keeps history; a leaked secret must be rotated | [secret-scan.yml](../../plugins/agent-harness/templates/.github/workflows/secret-scan.yml) |
| Instruction file limits, `paths:` present | A rule without `paths:` loads everywhere, silently | [instruction-files.yml](../../plugins/agent-harness/templates/.github/workflows/instruction-files.yml) |
| `paths:` globs match a file | A glob that matches nothing never loads | [pseudocode](../../plugins/agent-harness/reference/checks.md#paths-globs-match) |
| Paths in backticks exist | A pointer to a deleted file sends the agent nowhere | [pseudocode](../../plugins/agent-harness/reference/checks.md#backtick-paths-exist) |
| Commands in AGENTS.md exist | A command that does not exist teaches the agent to fail | [pseudocode](../../plugins/agent-harness/reference/checks.md#commands-exist) |
| Claims | Binds a line of an instruction file to a fact, so it cannot go stale | [pseudocode](../../plugins/agent-harness/reference/checks.md#claims), [claims.json](../../plugins/agent-harness/templates/.claude/claims.json) |
| Actions pinned by commit SHA | A tag can be moved to other code | [pseudocode](../../plugins/agent-harness/reference/checks.md#pinned-actions) |
| No `${{ }}` of author data in `run:` | A branch name or title in `run:` is code the author controls | [pseudocode](../../plugins/agent-harness/reference/checks.md#run-interpolation) |
| Comment conventions | History in comments rots | [pseudocode](../../plugins/agent-harness/reference/checks.md#comment-conventions) |

The ones marked pseudocode are left for you to port to your language, because a checker is code you will
maintain. The `audit` skill runs all of them by reading the repository, so you can start there.

## Claims

A claim says: "this line of `AGENTS.md` is true because this fact holds in the repository."

```json
{ "id": "testing-rule-exists", "source": "AGENTS.md",
  "quote": "- `.claude/rules/testing.md` - test conventions; ...",
  "type": "file_exists", "args": { "path": ".claude/rules/testing.md" } }
```

The checker fails when the quoted line disappears (update the claim) or the fact stops being true (update
the line or the code). Read the claims file from the base branch in CI, so a pull request cannot delete the
claim it breaks.

## Workflow hygiene

- Start every workflow with `permissions: {}` and grant each job only what it needs.
- `actions/checkout` with `persist-credentials: false` unless a later step pushes.
- Pin every third-party action by commit SHA, with the version in a comment, and let Dependabot update both
  ([dependabot.yml](../../plugins/agent-harness/templates/.github/dependabot.yml)).
- Pass untrusted values through `env:`, never `${{ }}` inside `run:`.
- `pull_request_target` runs with secrets and the base branch's workflow: never check out the pull
  request's code in it. The title check uses it safely because it reads the title as data.
- `concurrency` with `cancel-in-progress` saves minutes on pushes that follow each other.

## Locally, before CI

The same rules run earlier when they can: a pre-commit hook for secrets and formatting, a `commit-msg` hook
for commit format, and the agent's own local gates (the `ship-pr` skill runs them before opening a pull
request). CI is still the gate, because local hooks can be skipped.

## Sources

- GitHub Actions security hardening:
  https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- Certification guide 3.6, CI/CD integration:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-6-cicd-integration
