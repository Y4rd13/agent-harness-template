# 08. Pull request flow and release

The harness meets the team at the pull request. This lesson walks one change from a branch to a release,
and shows where each layer acts.

## The flow

```text
1. branch        feat/short-name from main; the hook refuses commits on main
2. implement     agent follows AGENTS.md and the path-scoped rules it loads
3. local gates   lint, format, type check, tests (the same commands CI runs)
4. self-review   the author reads the full diff; a reviewer subagent can help
5. open          /ship-pr: Conventional Commit title, template body, draft
6. CI            deterministic checks; AI review in shadow or advise mode
7. human review  CODEOWNERS for harness files; findings verified, not obeyed
8. merge         squash, with an explicit maintainer OK
9. release       the squash title decides the version bump
```

## The pull request template

[pull_request_template.md](../../plugins/agent-harness/templates/.github/pull_request_template.md) has:

- **a title table** mapping each Conventional Commit type to the release it causes;
- **Summary** and **Test plan**;
- **Author checklist**, labelled as a self-attestation: "I have read the full diff and can explain every
  change". No check enforces it; it is there so nobody can say they were not asked;
- **a generative-AI line**: one of two plain sentences, kept as plain text so it is not a checkbox to tick
  without reading;
- **Evidence** for user-visible or risky changes: a screenshot, a request and response, test output.

## Size

Large pull requests get skimmed, by people and by models. Split anything above about 1000 non-test lines or
crossing three or more layers, for example the data layer, then the service, then the interface, as stacked
pull requests. The AI review template enforces the line limit by refusing to review above it.

## Ownership

[CODEOWNERS](../../plugins/agent-harness/templates/.github/CODEOWNERS) covers the harness only: `AGENTS.md`,
`CLAUDE.md`, `.claude/` and `.github/`. A change to what every agent reads, or to the checks that enforce
it, needs an owner's review. Application code stays out, so ordinary pull requests do not wait on the
owners. Keep an instruction change in its own pull request when the feature does not need it.

## Review discipline

- A review finding, human or AI, is a claim. Verify it against the code before acting; decline the ones
  that do not hold, with the reason in the thread.
- Verify after each fix, not at the end. Most new findings on a pull request are regressions introduced by
  its own earlier fixes; before pushing, re-read the call sites and siblings of everything you touched.
- Group related fixes into one commit, and aim for two review rounds at most.
- Resolve threads as findings close, so what is still open is readable at a glance.

## Branch protection

Make the checks required on `main` with a ruleset or branch protection: the CI gate, the title check, the
secret scan, and later `ai-review/gate`. Require a pull request, and for harness paths a code owner review.
Two pull requests that each pass can still break `main` together (two migration heads, a lockfile
conflict): require branches to be up to date before merging, or use a merge queue.

## Release from the title

With squash merges, the pull request title becomes the commit on `main`. A release tool reads the commit
type:

| Title | Release |
| --- | --- |
| `fix(auth): handle an expired token` | patch |
| `feat(search): add filters` | minor |
| `feat!: drop the v1 API` | major |
| `docs: ...`, `chore: ...`, `ci: ...` | none |

semantic-release and release-please both work this way. That is why the title is checked before merge, and
why an agent must never pick a type casually.

A trap: GitHub skips push workflows when the commit message contains the skip marker (the words "skip ci"
in square brackets), in the title or the body. A squash body that quotes it, for example in a pull request
about the release process, silently turns off CI, release and deploy for that merge.

## Sources

- Conventional Commits: https://www.conventionalcommits.org
- GitHub, about code owners:
  https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
- GitHub, skipping workflow runs:
  https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/skipping-workflow-runs
