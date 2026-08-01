# relex-mcp

Generic MCP package for Relex. Skills and workflow guidance are agent-agnostic;
the live tools are hosted at `https://relex.legal/api/mcp`.

## Non-negotiables

- Do not invent a second MCP server — improve the backend `/v1/mcp` surface.
- Platform-specific install UX belongs in relex-claude / relex-gpt / relex-grok /
  relex-gemini; keep this repo host-neutral.
- PII claims must match backend enforcement.
- Bump `plugin/plugin.json` version on content changes.
