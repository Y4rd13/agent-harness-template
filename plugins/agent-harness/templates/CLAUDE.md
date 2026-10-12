@AGENTS.md

<!--
Template notes: this file is an import shim. Claude Code reads CLAUDE.md, and the line above pulls
AGENTS.md into context at launch. Recent Claude Code versions read AGENTS.md on their own when no
CLAUDE.md exists, but some sessions cannot, and the import never loads it twice, so keep it.
Keep this file under 40 lines: growth here means someone is duplicating AGENTS.md.
-->

## Claude-specific notes

- `AGENTS.md` (imported above) is the canonical conventions file; edit it there, not here. This file only carries what applies to Claude Code alone.
- Path-scoped rules in `.claude/rules/` load automatically when Claude touches matching files. Their one-line index is in AGENTS.md.
- <Claude-only tooling: a project skill such as `/ship-pr`, a plugin to install, an MCP server to enable.>
- Keep this file under 40 lines and AGENTS.md under 200.
