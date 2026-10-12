# agent-harness-template

[Español](README.es.md)

Templates, a Claude Code plugin and short lessons for making an AI coding assistant follow your
repository's conventions, with checks that do not depend on it remembering them.

The repository is mostly Markdown. It holds almost no code: where a check needs logic, you get pseudocode
to port, and code appears only where nothing works without it (one hook, one shell step, one review gate).

## The idea: layers, from advice to enforcement

```text
  advice                                                                    enforcement
  ------------------------------------------------------------------------------------>
  AGENTS.md          .claude/rules/      skills          settings + hooks   CI checks         AI review gate     human
  CLAUDE.md          (path-scoped)       subagents       (client-side)      (deterministic)   (model + code)     review
  every session      when a path         on request      every tool call    every PR          every PR           merge
                     is touched
```

- **Instruction files** tell the agent how the repository works. It usually follows them; nothing forces it.
- **Settings and hooks** are applied by Claude Code whatever the model decides.
- **CI** fails the pull request for everyone: humans, any agent, any tool.
- **The AI review** catches what no rule states, inside a sandbox, and only blocks on evidence a script
  verified.
- **A person** owns the merge.

Rule of thumb: if a machine can check it, write the check; if it must always hold, use a hook or a required
check; only what is left goes into prose.

## Quickstart A: let the plugin set it up

```bash
claude plugin marketplace add <owner>/agent-harness-template
claude plugin install agent-harness@agent-harness-template
```

Then, inside Claude Code in your repository:

| Command | What it does |
| --- | --- |
| `/agent-harness:init dry-run` | Reads your repository and prints the harness it would create, with your real commands. Writes nothing |
| `/agent-harness:init` | Asks what it cannot read, shows every file, and writes only what you approve |
| `/agent-harness:audit` | Checks an existing harness: limits, rules that load everywhere, stale paths, unpinned actions, contradictions |
| `/agent-harness:add-rule <instruction>` | Decides where a new instruction belongs (check, hook, skill, rule, AGENTS.md or a doc) and proposes the change |

## Quickstart B: copy by hand

1. Copy [AGENTS.md](plugins/agent-harness/templates/AGENTS.md) and
   [CLAUDE.md](plugins/agent-harness/templates/CLAUDE.md) to your repository root and replace every
   `<...>`. Delete the lines that do not apply rather than leaving placeholders.
2. Add one rule per part of the tree with its own conventions, from
   [rules/](plugins/agent-harness/templates/.claude/rules/testing.md), each with a `paths:` that matches your
   files.
3. Copy [settings.json](plugins/agent-harness/templates/.claude/settings.json) to `.claude/` and fill the
   allow list with your commands.
   The hook needs `jq` on every machine; without it, it blocks every Bash call and says why.
4. Copy [docs/conventions/](plugins/agent-harness/templates/docs/conventions/agent-instructions.md) too:
   `AGENTS.md` points to both files in it.
5. Copy the workflows you want from
   [.github/workflows/](plugins/agent-harness/templates/.github/workflows/ci.yml), the
   [PR template](plugins/agent-harness/templates/.github/pull_request_template.md) and
   [CODEOWNERS](plugins/agent-harness/templates/.github/CODEOWNERS). For the AI review, merge
   [ai-review.prompt.md](plugins/agent-harness/templates/.github/prompts/ai-review.prompt.md) and
   [ai-review.schema.json](plugins/agent-harness/templates/.github/ai-review.schema.json) first, in their
   own pull request: the workflow reads them from the base branch.
6. If your default branch is not `main`, replace `main` in the hook, in the workflows' `branches:` and in
   `AGENTS.md`.
7. Add `CLAUDE.local.md` and `.claude/settings.local.json` to `.gitignore`. If your global gitignore excludes
   `.claude/`, add `!.claude/` to the repository's `.gitignore`.
8. Open Claude Code and run `/memory` to confirm `CLAUDE.md` and `AGENTS.md` loaded.

## What is in the templates

| Template | What it does | When it loads or runs |
| --- | --- | --- |
| [AGENTS.md](plugins/agent-harness/templates/AGENTS.md) | Commands, stack, architecture, Critical Invariants, boundaries. Canonical for every agent | Every session |
| [CLAUDE.md](plugins/agent-harness/templates/CLAUDE.md) | Imports `AGENTS.md`, plus Claude-only notes | Every Claude Code session |
| [CLAUDE.local.md.example](plugins/agent-harness/templates/CLAUDE.local.md.example) | Personal notes, never committed | Every session, for you only |
| [.claude/rules/](plugins/agent-harness/templates/.claude/rules/layer.md) | Conventions for one part of the tree | When a matching file is read or edited |
| [.claude/settings.json](plugins/agent-harness/templates/.claude/settings.json) | Permissions and a hook that blocks commits and pushes to `main` | Every tool call |
| [ship-pr skill](plugins/agent-harness/templates/.claude/skills/ship-pr/SKILL.md) | Local gates, Conventional Commit title, draft pull request, never merges | When you run `/ship-pr` |
| [reviewer subagent](plugins/agent-harness/templates/.claude/agents/reviewer.md) | Read-only review of a diff against the conventions | When delegated to |
| [.mcp.json.example](plugins/agent-harness/templates/.mcp.json.example) | MCP servers with credentials from the environment | Every session |
| [agent-instructions.md](plugins/agent-harness/templates/docs/conventions/agent-instructions.md) | What goes where, and when a change to the instructions is justified | When opened |
| [comments.md](plugins/agent-harness/templates/docs/conventions/comments.md) | WHY not WHAT, no history in code | When opened |
| [user-level/](plugins/agent-harness/templates/user-level/CLAUDE.md) | Your personal `~/.claude/` layer | Every session, every project |
| [ci.yml](plugins/agent-harness/templates/.github/workflows/ci.yml) | One gate job: lint, format, types, tests, build | Every pull request |
| [pr-title.yml](plugins/agent-harness/templates/.github/workflows/pr-title.yml) | Conventional Commit title | Every pull request |
| [secret-scan.yml](plugins/agent-harness/templates/.github/workflows/secret-scan.yml) | gitleaks over the pull request's commits | Every pull request |
| [instruction-files.yml](plugins/agent-harness/templates/.github/workflows/instruction-files.yml) | Instruction file limits and `paths:` | Every pull request |
| [ai-review.yml](plugins/agent-harness/templates/.github/workflows/ai-review.yml) | Sandboxed model review plus a gate that verifies quotes, modes off, shadow, advise | Every ready pull request |
| [pull_request_template.md](plugins/agent-harness/templates/.github/pull_request_template.md) | Summary, test plan, author checklist, AI disclosure, evidence | Every pull request |
| [CODEOWNERS](plugins/agent-harness/templates/.github/CODEOWNERS) | Owner review for the harness files only | Every pull request |
| [dependabot.yml](plugins/agent-harness/templates/.github/dependabot.yml) | Grouped weekly updates, actions included | Weekly |
| [claims.json](plugins/agent-harness/templates/.claude/claims.json) | Lines of the instructions bound to facts a check verifies | When you run the claims check |
| [checks.md](plugins/agent-harness/reference/checks.md) | Pseudocode for every harness check | When you port one, or run `audit` |

## Lessons

1. [The memory hierarchy: CLAUDE.md, AGENTS.md and friends](docs/learn/01-memory-hierarchy.md)
2. [Path-scoped rules](docs/learn/02-path-scoped-rules.md)
3. [Skills, commands, subagents and plugins](docs/learn/03-skills-commands-plugins.md)
4. [Settings, permissions and hooks](docs/learn/04-settings-permissions-hooks.md)
5. [MCP servers](docs/learn/05-mcp.md)
6. [The deterministic CI layer](docs/learn/06-deterministic-ci.md)
7. [The LLM review layer](docs/learn/07-llm-review-layer.md)
8. [Pull request flow and release](docs/learn/08-pr-flow-and-release.md)
9. [Maintaining the instructions](docs/learn/09-maintaining-instructions.md)

And [Lessons learned](docs/lessons-learned.md): what went wrong on real repositories, and what to do.

## Repository layout

```text
.claude-plugin/marketplace.json        this repository is a Claude Code marketplace
plugins/agent-harness/
  .claude-plugin/plugin.json
  skills/init|audit|add-rule/SKILL.md  the three skills
  templates/                           every file the skills copy, usable by hand too
  reference/checks.md                  pseudocode for the harness checks
docs/learn/                            the lessons
docs/lessons-learned.md
docs/es/                               Spanish translation
```

## Credits and sources

- Claude Code documentation: https://code.claude.com/docs
- The `AGENTS.md` format: https://agents.md
- Claude Certification Guide, an independent study resource whose lessons the guides here link to:
  https://claudecertificationguide.com
- GitHub Actions security hardening:
  https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions

## License

[MIT](LICENSE)
