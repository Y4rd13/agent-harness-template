# AGENTS.md

<!--
Template notes (block-level HTML comments are stripped before Claude Code sends this file to the model,
so these notes cost no context):
- This is the canonical conventions file for every coding agent and every human. CLAUDE.md imports it.
- Keep it under 200 lines. Every line is read at the start of every session.
- Per line, ask: "Would removing this cause an agent to make a mistake?" If not, cut it.
- Rules a machine can check belong in CI or a hook. Here, name the check instead of restating the rule.
- Replace every <...> placeholder. Delete a section only if it truly does not apply.
-->

<One sentence: what this repository is and who uses it.> This is the canonical conventions file for every coding agent and human. `CLAUDE.md` imports it; edit here, never maintain two copies. Rules a machine can check live in CI (see CI checks), not in prose.

## Commands

<!-- Only commands an agent cannot guess. Include the one-time setup and the single-test form. -->

```bash
<install command>          # One-time setup
<dev command>              # Start the dev server
<test command>             # Run the full test suite
<single test command>      # Run one test file
<lint command>             # Lint
<format command>           # Format (run before every push; CI checks it strictly)
<typecheck command>        # Type check
<build command>            # Production build
```

## Stack

<!-- Language and framework versions that change how code is written. Not every dependency. -->

- <Language + version>, <framework + version>
- <Data layer: database, ORM, migration tool>
- <Testing: runner + libraries>; tooling: <linter, formatter, package manager>

## Architecture rules

<!-- The layering and the decisions an agent cannot infer from one file. -->

Layered: `<entry layer>` -> `<service layer>` -> `<data layer>`, imports flow downward only. Before creating a new module, check whether an existing one already owns the concern.

- <Rule an agent would get wrong without being told, with the path that shows the pattern.>
- <When to use pattern A versus pattern B; a small table works well here.>
- Comments follow `docs/conventions/comments.md`: WHY, not WHAT; no ticket keys or history in code.

## Critical Invariants

<!--
Security and data rules every agent and reviewer must see. Violating one is a blocking review finding.
Keep this section here, not in .claude/rules/: other agents do not read Claude's path-scoped rules.
Each line: the rule, the exception if any, and where it is enforced.
-->

Violating any of these is a blocking review finding:

- **<Authorization invariant>**, for example: every query on user-owned data filters by the owner. Exceptions: <list them, with the function that implements each>.
- **No secrets, tokens, or personal data in logs, responses, error messages, or anything posted to a pull request or ticket.**
- **<Data integrity invariant>**, for example: migrations are reversible and the main branch has exactly one migration head.
- **No breaking change to a public interface** without a migration path or a new version.

## Boundaries

**Always**

- Work on a feature branch and integrate through a pull request to `main`. Required checks: <list the checks your branch protection requires>.
- Keep pull requests reviewable: split anything above about 1000 non-test lines or crossing three or more layers; stacked pull requests are welcome.
- Use Conventional Commit pull request titles: `feat(scope): add X`. The squash-merged title drives the release, so the type decides the version bump.
- Run `<format command>` and `<lint command>` before pushing.

**Ask first**

- Adding a dependency, changing a public interface, or touching <a risky area: auth, billing, migrations>.
- Anything that mutates shared infrastructure.

**Never**

- Push or commit directly to `main`; merge without an explicit maintainer OK.
- Commit secrets of any kind, including development credentials. Git keeps history, so a leak forces a rotation.
- Put ticket keys, pull request numbers, dates, or measurements in code comments.
- Delete or weaken a failing check to get green. Fix the root cause or raise it with a maintainer.

## Assisted, not autopiloted

The developer owns every change; you assist. Before a pull request leaves draft, make sure they have seen the diff and can explain what it does and why; walk them through anything non-obvious, unprompted. Never open a pull request or mark one ready for review without an explicit instruction. When a change is user-visible or touches a risky path, remind them to attach evidence to the pull request body: a screenshot, a request and response, or test output.

AI assistance never replaces self-review: the author reads their own full diff, and a reviewer who uses AI owns the verdict and verifies each finding against the code before posting it.

## CI checks (what blocks a merge; do not restate in reviews)

<!-- Name the required checks and what each covers in one line. The full list of workflows belongs in README.md, because it changes often. -->

<N> checks block a merge: `<check name>` (<what it runs>), `<check name>` (<what it runs>). A review does not restate what a check already reports.

## Detail on demand (one line each; read when relevant)

<!-- One line per document: what it holds and when to read it. Path-scoped rules are listed here so every agent knows they exist. -->

- `.claude/rules/testing.md` - test conventions; read before touching tests (Claude Code loads it automatically)
- `.claude/rules/<layer>.md` - rules for `<layer path>`; read before touching it (Claude Code loads it automatically)
- `docs/conventions/agent-instructions.md` - what goes in `AGENTS.md`, `CLAUDE.md`, `.claude/rules/` and CI; read before editing any of them
- `docs/conventions/comments.md` - the comments guide
- `<docs/runbook-x.md>` - <operation it covers>
