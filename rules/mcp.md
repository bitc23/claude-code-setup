# MCP — Model Context Protocol

MCP servers expose external capabilities (filesystems, databases, browsers, APIs) to Claude as tools. Each server runs as a subprocess (stdio) or over HTTP and registers a set of tools the model can call.

## Where Configs Live

- `<project>/.mcp.json` — project-scoped, committed to the repo (treat as code review surface)
- `~/.mcp.json` — user-scoped, applies to every session
- `--mcp-config <path>` — load a specific file for one session

Project config takes precedence over user config when both define the same server name.

## Security Posture

**Treat MCP servers as third-party code with full access to whatever you give them.** Before adding a server:

- Read the source. If you can't, don't add it
- Restrict the surface (read-only filesystem mounts, scoped API tokens)
- Prefer official servers from `modelcontextprotocol.io` over random GitHub repos
- Never commit a server config that contains a real token or key — use environment variable references

## Minimal Example

`<project>/.mcp.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/stephanbuechler/Developer/this-project"],
      "env": {}
    }
  }
}
```

This gives Claude read/write access to one specific directory through the official filesystem server.

## Common Server Types

| Kind | What it gives Claude | Risk |
|------|---------------------|------|
| Filesystem | Read/write/list within a scoped path | Medium — scope tightly |
| GitHub | Issue/PR/repo operations via the GitHub API | Medium — token permissions matter |
| Postgres / SQLite | Query a database | High — read-only role recommended |
| Browser (Puppeteer / Playwright) | Drive a real browser | High — can navigate to anything |
| Slack / Linear | Read/post in your team's spaces | High — can spam, leak channels |

## Inspecting What's Connected

In-session, run `/mcp` to see active servers and the tools each one exposes. Useful when:

- A tool isn't behaving the way the docs say
- You want to confirm a server actually loaded after a config change
- You're auditing what surface area Claude has

## Personal-Project Defaults

For my personal repos I usually run **no MCP servers**. The local Bash/Read/Edit/Write tools cover almost everything, and adding an MCP server is more attack surface for marginal benefit. Add a server only when you have a concrete recurring need it solves better than a slash command + bash.
