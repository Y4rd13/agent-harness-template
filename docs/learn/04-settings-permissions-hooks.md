# 04. Settings, permissions and hooks

Instruction files are advice: the model reads them and usually follows them. Settings and hooks are
enforcement: Claude Code applies them whatever the model decides. Anything that must hold every time, with
no exception, belongs here.

## Settings files

| File | Shared | Use |
| --- | --- | --- |
| Managed settings (set by IT) | Everyone | Policy no one can override |
| `.claude/settings.local.json` | Only you, gitignored | Your local overrides for this project |
| `.claude/settings.json` | The team, through git | The project's permissions and hooks |
| `~/.claude/settings.json` | Only you, every project | Your defaults |

Precedence, highest first: managed, then command-line flags, then local, then project, then user. Deny
rules are special: a deny at any level blocks the call, even if another level allows it.

Template: [settings.json](../../plugins/agent-harness/templates/.claude/settings.json).

## Permissions

```json
{
  "permissions": {
    "allow": ["Bash(npm test)", "Bash(npm run lint)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  }
}
```

- `allow` runs without asking, `ask` always prompts, `deny` refuses.
- Deny is evaluated first: an allow rule cannot carve an exception out of a deny rule.
- Bash rules match the command text by prefix with `*` wildcards. That makes them easy to sidestep:
  `Bash(git push *)` does not match `git -C . push origin main`. Use them for convenience and as a first
  line, not as your only guard for something dangerous.
- `Read` deny rules are how you keep secrets out of the model's context.
- A project's `allow` rules apply only after you accept the workspace trust dialog for that folder.

## Hooks

A hook is a command Claude Code runs at a point in its loop. It is code, so a prompt cannot talk it out of
its decision.

| Event | Runs | Can block |
| --- | --- | --- |
| `PreToolUse` | before a tool call | yes: exit code 2, or a `deny` decision |
| `PostToolUse` | after a tool call | no: the action already happened. Use it to check or format |
| `UserPromptSubmit` | before your prompt reaches the model | yes |
| `SessionStart` | when a session starts | no; it can add context |

The hook receives the event as JSON on stdin. Exit code 2 blocks the action and sends stderr back to
Claude, so it can correct course. Other non-zero exits are reported as hook errors and the action goes
ahead.

A blocking hook wins over an allow rule. It does not bypass deny or ask rules: those still apply.

### The template's hook

The template blocks commits and pushes to `main`. In pseudocode:

```text
command = stdin.tool_input.command
branch  = git branch --show-current
if command runs "git ... push" with a refspec that is main (main, HEAD:main), as a whole word:
    block "no commits or pushes to main"
if branch is main and command runs "git ... commit", or a bare "git push" / "git push <remote>":
    block "no commits or pushes to main"
otherwise allow
```

It ships as one line of shell in `settings.json`, because a hook needs code to run. It also matches
`git -C <dir> push`, which the permission rule above misses.

## When to use which

| You need | Use |
| --- | --- |
| Convenience: stop being asked about safe commands | `allow` rules |
| A second look before something risky | `ask` rules |
| Keep files out of the model's reach | `Read` deny rules |
| A rule that must hold even when the command is written oddly | a `PreToolUse` hook |
| A rule for everyone, including humans and other agents | a CI check (lesson 06) |

A rule of thumb from the certification guide: if one failure would cost real money or data, use a hook,
not an instruction.

## Common mistakes

- Blocking in `PostToolUse`: the action already ran.
- Trusting a Bash permission rule to stop a determined command: it matches text by prefix.
- A hook that fails silently (wrong path, missing `jq`): the action goes ahead. Test hooks with sample JSON
  on stdin before relying on them.
- Committing `.claude/settings.local.json`: it is personal; keep it in `.gitignore`.

## Sources

- Claude Code docs, settings: https://code.claude.com/docs/en/settings
- Claude Code docs, permissions: https://code.claude.com/docs/en/permissions
- Claude Code docs, hooks: https://code.claude.com/docs/en/hooks
- Certification guide, domain 3 quick reference:
  https://claudecertificationguide.com/learn/quick-reference/domain-3
- Certification guide 1.5, hooks:
  https://claudecertificationguide.com/learn/1-agentic-architecture/1-5-agent-sdk-hooks
