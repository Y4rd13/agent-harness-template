---
name: ship-pr
description: Open a draft pull request from the current local work, following this repository's conventions. Use when the user says "ship this", "open a PR", "create the pull request" or "send this for review". Never merges.
disable-model-invocation: true
argument-hint: "[ready]"
---

# Ship a pull request

<!-- A project skill: a procedure of several steps that runs on request. Its description is always in context; this body loads only when the skill is invoked. Save it as .claude/skills/ship-pr/SKILL.md and run it as /ship-pr. -->

Open a pull request for the work on the current branch. Stop and tell the user at the first step that fails; never work around it.

1. **Branch.** If the current branch is the default branch, create a feature branch first
   (`feat/`, `fix/`, `chore/`, `docs/` or `refactor/` plus a kebab-case name). Never commit to the default branch.
2. **Local gates.** Run the commands listed in `AGENTS.md` under Commands for lint, format, type check and tests.
   Commit what the formatter rewrote. Every gate must pass.
3. **Scope.** Show `git diff --stat` against the default branch. If the change passes about 1000 non-test lines or
   crosses three or more layers, propose a split into stacked pull requests before going on.
4. **Secrets.** Scan the diff for anything shaped like a credential (keys, tokens, passwords, connection strings).
   Stop if one appears, even a development credential.
5. **Title.** Write a Conventional Commit title, `<type>(<scope>): <summary>`. The type decides the release bump:
   `feat` is minor, `fix` is patch, `feat!` is major, the rest release nothing.
6. **Body.** Fill `.github/pull_request_template.md`: Summary, Test plan, and Evidence when the change is
   user-visible or touches a risky path. Leave the Author checklist for the author to tick.
7. **Open it.** Push the branch and run `gh pr create --draft` with that title and body. With the argument `ready`,
   open it ready for review instead, only because the user asked for it.
8. **Report.** Print the pull request URL. Remind the author to read the full diff before it leaves draft.

Never merge, never push to the default branch, and never mark a draft ready unless the user asked.
