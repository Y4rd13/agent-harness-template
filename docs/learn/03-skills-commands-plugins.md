# 03. Skills, commands, subagents and plugins

Instruction files describe how the repository works. Skills describe how to do a job: open a pull
request, review one, set up a harness. A skill costs almost nothing until it is used, so procedures belong
there and not in `CLAUDE.md`.

## Skills

A skill is a directory with a `SKILL.md`:

```text
.claude/skills/ship-pr/SKILL.md      ->  /ship-pr          (project, shared through git)
~/.claude/skills/ship-pr/SKILL.md    ->  /ship-pr          (personal)
<plugin>/skills/init/SKILL.md        ->  /<plugin>:init    (from a plugin)
```

Only the `description` is always in context, so Claude knows the skill exists. The body loads when the
skill runs, either because you typed `/ship-pr` or because Claude decided the description matched.

Frontmatter fields worth knowing:

| Field | What it does |
| --- | --- |
| `description` | What the skill does and when to use it. Put the trigger first; Claude matches on it |
| `when_to_use` | Extra trigger phrases, appended to the description |
| `disable-model-invocation: true` | Only you can run it, with `/name`. Use it for anything that writes, pushes or spends |
| `allowed-tools` | Tools Claude may use **without asking** while the skill runs. It pre-approves; it does not restrict |
| `argument-hint` | The hint shown in autocomplete, such as `[dry-run]` |
| `context: fork` | Run the skill in a separate subagent context, so its output does not fill your session |

Inside a skill, `${CLAUDE_SKILL_DIR}` is the skill's directory, and in a plugin `${CLAUDE_PLUGIN_ROOT}` is
the plugin's. Use them to read files that ship with the skill; this plugin's skills read their templates
that way.

Template: [ship-pr/SKILL.md](../../plugins/agent-harness/templates/.claude/skills/ship-pr/SKILL.md).

### Commands

`.claude/commands/<name>.md` is the older form of a skill and still works: it also creates `/<name>`. A
skill and a command with the same name: the skill wins. Prefer skills for new work: they can bundle files.
A loose `.md` file directly inside `.claude/skills/` does not create anything; it must be
`skills/<name>/SKILL.md`.

## Subagents

A subagent is a Markdown file in `.claude/agents/` with its own instructions and its own context window.
The main session hands it a task and gets back a summary, so a long search or review does not crowd the
main context. Its `tools` field limits what it can use: a reviewer with `Read, Grep, Glob` cannot edit the
code it reviews.

Template: [reviewer.md](../../plugins/agent-harness/templates/.claude/agents/reviewer.md).

## Plugins

A plugin packages skills, subagents, hooks and MCP servers so a team installs them with one command and
gets updates. A marketplace is a repository with `.claude-plugin/marketplace.json` that lists plugins. This
repository is both:

```text
.claude-plugin/marketplace.json            the catalog: one plugin, "agent-harness"
plugins/agent-harness/.claude-plugin/plugin.json
plugins/agent-harness/skills/{init,audit,add-rule}/SKILL.md
plugins/agent-harness/templates/           files the skills copy
plugins/agent-harness/reference/checks.md  pseudocode the audit skill runs
```

Install it:

```bash
claude plugin marketplace add <owner>/agent-harness-template
claude plugin install agent-harness@agent-harness-template
```

Develop one without installing: `claude --plugin-dir ./plugins/agent-harness`, and check it with
`claude plugin validate <path>`. Keep the entry name in `marketplace.json` and the `name` in `plugin.json`
the same.

## What goes where

| You want | Use |
| --- | --- |
| A fact true in every session | `AGENTS.md` |
| A procedure run on request | a skill |
| Heavy reading or review that should not fill your context | a subagent |
| To share skills, agents and hooks across repositories | a plugin |
| Something that must always happen | a hook (lesson 04), not a skill |

## Exam vs current behavior

The certification guide marks two answers where the exam guide and current Claude Code differ. The exam
guide describes `allowed-tools` as restricting tools; current Claude Code treats it as pre-approval, and
restricting is done with `disallowed-tools` or deny rules. The exam guide also describes `argument-hint` as
prompting for missing arguments; today it is an autocomplete hint. The guide also notes that the exam
calls the subagent tool `Task`, which Claude Code now calls `Agent`.

## Sources

- Claude Code docs, skills: https://code.claude.com/docs/en/skills
- Claude Code docs, subagents: https://code.claude.com/docs/en/sub-agents
- Claude Code docs, plugins: https://code.claude.com/docs/en/plugins/create and
  https://code.claude.com/docs/en/plugin-marketplaces
- Certification guide 3.2, slash commands and skills:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-2-slash-commands-skills
- Certification guide 1.3, subagents:
  https://claudecertificationguide.com/learn/1-agentic-architecture/1-3-subagent-invocation-context
