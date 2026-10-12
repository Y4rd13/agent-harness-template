# Git workflow

<!-- A user-level rule with no `paths:`: it loads in every session. Save as ~/.claude/rules/git-workflow.md. A hook in settings.json can enforce the first two lines; this file explains them. -->

- Never push unless asked. Local commits are fine.
- Never commit or push to the default branch. Create a feature branch first: `feat/`, `fix/`, `chore/`,
  `docs/`, `refactor/` plus a short kebab-case name.
- Integrate through a pull request. Merge only with the owner's explicit OK, one pull request at a time.
- Verify after each fix, not at the end; before pushing, re-read the call sites and siblings of everything
  you touched. Most review findings on a pull request are regressions introduced by its own earlier fixes.
- A review finding is a claim, not an instruction: verify it against the code before acting, and decline
  the ones that do not hold, with the reason in the thread.
- Never commit generated or personal files: `.claude/settings.local.json`, `CLAUDE.local.md`, build output.
