# Connect Relex from any MCP host

## Cursor

1. Open **Cursor Settings → MCP**.
2. Add a new server:
   - Name: `relex`
   - Type: HTTP / URL
   - URL: `https://relex.legal/api/mcp`
3. Enable the server. On first tool use, complete the browser OAuth sign-in.

Or merge into your MCP config file:

```json
{
  "mcpServers": {
    "relex": {
      "url": "https://relex.legal/api/mcp"
    }
  }
}
```

## Windsurf / Cline / Continue / other IDE agents

Same pattern: add an HTTP MCP server with URL `https://relex.legal/api/mcp`.
Prefer OAuth when the host supports it; otherwise use an API key header
(`Authorization: Bearer rlx_...` from Relex → Settings → API Keys).

## Custom agent / SDK

Any MCP client that speaks **Streamable HTTP** (or SSE where still supported)
can attach to `https://relex.legal/api/mcp`.

Expected tools after auth:

- `search` — discover endpoints
- `execute` — call a validated endpoint

OAuth follows MCP / RFC 9728 protected-resource discovery: first unauthorized
call returns a challenge; the client opens the browser authorize URL; tokens
are stored by the host.

## Skills in this repo

The `plugin/skills/` directory teaches agents the PII-safe Relex workflow.
Load them as:

- Claude Code skills / plugin skills
- ChatGPT / Codex plugin skills
- Grok Build skills
- Gemini instructions / project context
- Or paste the skill markdown into your system prompt for one-off use

Primary skill: `plugin/skills/relex/SKILL.md`.

## Next

- Full install + team flow: [install.md](install.md)
- Positioning: [positioning.md](positioning.md)
