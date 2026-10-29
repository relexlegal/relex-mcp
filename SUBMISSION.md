# Submission — Official MCP Registry (registry.modelcontextprotocol.io)

> Prepared 2026-10-28; updated the same day for MCP server version 1.1.0
> (branch `docs/read-api-1.1.0`). Nothing has been submitted. The backend with
> neutral server instructions, data-only responses and the `read_api` tool is
> on `master` (5a5d7b9) and deployed as revision `relex-backend-00192`.

Hosted server: `https://relex.legal/api/mcp` (streamable HTTP, OAuth 2.1 with
PKCE and dynamic client registration). Thirteen tools: `list_matters`,
`read_matter_context`, `diagnose_matter_sources`, `read_matter_source`,
`save_matter_work_product`, `correct_matter_ontology`, `conclude_matter_session`,
`find_legal_professionals`, `read_legal_professional`,
`prepare_professional_request`, `search`, `read_api`, `execute`.

Server version 1.1.0. `read_api` reads one GET endpoint (read-only; parameters
`path` and optional `query`). `execute` creates, updates or deletes through one
endpoint (POST/PUT/PATCH/DELETE); DELETE and other irreversible routes are not
applied by the tool and return a Relex deep link where the user confirms the
change.

## Done in this branch
- [x] `server.json` (schema 2025-12-11, validated): name `legal.relex/relex`, remote `streamable-http` → `https://relex.legal/api/mcp`, version 0.1.10, icon, repository.
- [x] README: 13-row tool table matching the server (including `read_api`); SECURITY/positioning: reading default corrected. Plugin 0.1.10.

## User must do
- [ ] Install `mcp-publisher` (github.com/modelcontextprotocol/registry releases).
- [ ] Prove ownership of relex.legal (the `legal.relex/*` namespace): either `mcp-publisher login dns --domain relex.legal --private-key <key>` with the TXT record `v=MCPv1; k=ed25519; p=<public key>` on relex.legal, or `mcp-publisher login http` with `/.well-known/mcp-registry-auth` served from relex.legal. Keep the private key out of git.
  (Alternative without DNS: rename to `io.github.relexlegal/relex` and use `mcp-publisher login github`.)
- [ ] `mcp-publisher validate` then `mcp-publisher publish`. Bump `version` for every later publish. The registry is in preview; one URL can belong to only one server name.
