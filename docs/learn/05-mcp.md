# 05. MCP servers

MCP (Model Context Protocol) servers give the agent tools beyond the built-in ones: your issue tracker, a
database, a browser, a documentation index. They are powerful and they cost context, so add them on
purpose.

## Where they are configured

| Scope | Stored in | Shared | Use |
| --- | --- | --- | --- |
| Local (the default) | `~/.claude.json`, under this project | Only you, this project | Trying a server, personal credentials |
| Project | `.mcp.json` at the repository root | The team, through git | Servers the whole team uses |
| User | `~/.claude.json` | Only you, every project | Your personal tools |

Add one with `claude mcp add <name> --scope project ...`, or write `.mcp.json` by hand.

Template: [.mcp.json.example](../../plugins/agent-harness/templates/.mcp.json.example).

## The format

```json
{
  "mcpServers": {
    "docs": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "headers": { "Authorization": "Bearer ${DOCS_MCP_TOKEN}" }
    },
    "local-db": {
      "command": "npx",
      "args": ["-y", "<package>@<exact version>"],
      "env": { "DATABASE_URL": "${DATABASE_URL:-postgresql://localhost:5432/dev}" }
    }
  }
}
```

- A remote server needs `"type": "http"`. An entry with a `url` and no `type` is read as a local (stdio)
  server and fails.
- `${VAR}` and `${VAR:-default}` expand from the environment. A missing variable with no default produces
  a warning and the server loads with the text unexpanded.
- **Never commit a credential in `.mcp.json`.** Reference an environment variable instead.
- Pin the version of a server you run with `npx` or `uvx`, as you pin any other dependency.

## Good practice

- Try a community or vendor server before writing your own.
- Each server's tools and descriptions take context. Keep the project list short, and put rarely used
  servers at your local scope.
- Tool descriptions matter: a vague one loses to the built-in tools and is never used.
- Check status with `/mcp` in a session, or `claude mcp list` in the shell.
- In CI, start the agent with `--strict-mcp-config` and an explicit list (or none), so a pull request
  cannot add a server to the review run (lesson 07).

## Exam vs current behavior

The certification guide notes that the exam describes two scopes, project (`.mcp.json`) and user
(`~/.claude.json`), while current Claude Code has three: local (the default), project and user.

## Sources

- Claude Code docs, MCP: https://code.claude.com/docs/en/mcp
- Certification guide 2.4, MCP server integration:
  https://claudecertificationguide.com/learn/2-tool-design-mcp/2-4-mcp-server-integration
