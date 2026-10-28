# Submission — Official MCP Registry (registry.modelcontextprotocol.io)

> Prepared 2026-10-28 on branch `directory-review-2026-10`. Nothing has been
> submitted, pushed or deployed. Before any submission: deploy the backend branch
> `directory-review-2026-10` (neutral server instructions, data-only responses),
> then merge and push this repo.

Hosted server: `https://relex.legal/api/mcp` (streamable HTTP, OAuth 2.1 with
PKCE and dynamic client registration). Twelve tools: `list_matters`,
`read_matter_context`, `diagnose_matter_sources`, `read_matter_source`,
`save_matter_work_product`, `correct_matter_ontology`, `conclude_matter_session`,
`find_legal_professionals`, `read_legal_professional`,
`prepare_professional_request`, `search`, `execute`.

## Done in this branch
- [x] `server.json` (schema 2025-12-11, validated): name `legal.relex/relex`, remote `streamable-http` → `https://relex.legal/api/mcp`, version 0.1.8, icon, repository.
- [x] README: 12-row tool table matching the server; SECURITY/positioning: reading default corrected. Plugin 0.1.8.

## User must do
- [ ] Deploy backend; merge and push this repo.
- [ ] Install `mcp-publisher` (github.com/modelcontextprotocol/registry releases).
- [ ] Prove ownership of relex.legal (the `legal.relex/*` namespace): either `mcp-publisher login dns --domain relex.legal --private-key <key>` with the TXT record `v=MCPv1; k=ed25519; p=<public key>` on relex.legal, or `mcp-publisher login http` with `/.well-known/mcp-registry-auth` served from relex.legal. Keep the private key out of git.
  (Alternative without DNS: rename to `io.github.relexlegal/relex` and use `mcp-publisher login github`.)
- [ ] `mcp-publisher validate` then `mcp-publisher publish`. Bump `version` for every later publish. The registry is in preview; one URL can belong to only one server name.
