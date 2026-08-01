# OAuth for generic MCP hosts

How any MCP client authenticates to Relex without a special SDK.

## Endpoints

| Piece | URL |
|-------|-----|
| MCP resource | `https://relex.legal/api/mcp` |
| Protected Resource Metadata (RFC 9728) | `https://relex.legal/.well-known/oauth-protected-resource` |
| Authorization Server Metadata (RFC 8414) | `https://relex.legal/.well-known/oauth-authorization-server` |
| Dynamic client registration (RFC 7591) | `POST https://relex.legal/api/oauth/register` |
| Authorize | `https://relex.legal/oauth/authorize` |
| Token | `POST https://relex.legal/api/oauth/token` |
| Skill text for agents | `https://relex.legal/auth.md` |

## Flow

1. Host `POST`s MCP without `Authorization`.
2. Server returns **401** with:
   ```http
   WWW-Authenticate: Bearer resource_metadata="https://relex.legal/.well-known/oauth-protected-resource",
     scope="relex.agent relex.cases.read relex.cases.write relex.draft"
   ```
3. Host fetches PRM → discovers authorization server `https://relex.legal`.
4. Host fetches AS metadata → authorize, token, register endpoints; **PKCE S256** required; public client (`token_endpoint_auth_method: none`).
5. Host registers (dynamic) or uses Client ID Metadata Document.
6. Browser: user signs in with Google/Apple, approves scopes.
7. Host exchanges `code` + `code_verifier` for access (and refresh) token.
8. Host calls MCP with `Authorization: Bearer <access_token>`.
9. Tools: `search`, `execute`.

## Host requirements

- Public HTTPS reachability to `relex.legal`
- Streamable HTTP MCP client
- Honor `WWW-Authenticate` + `resource_metadata` (not only JSON error body)
- Local browser for PKCE redirect
- Secure per-user token storage

## Headless fallback

Relex **Settings → API Keys** → `Authorization: Bearer rlx_...`  
No browser; suitable for CI only. Prefer OAuth for humans.

## Scopes

- `relex.cases.read` / `relex.cases.write` / `relex.draft` / `relex.agent`

Tokens never unlock plaintext PII; those calls return deep links.

## Product UI names (same protocol)

| Product | UI label |
|---------|----------|
| Claude | Custom **connector** |
| ChatGPT | **App** / custom MCP connector |
| Grok.com | **Connector** (Custom) |
| Grok Build | **MCP server** / plugin |
| Gemini CLI | **MCP server** |
| Gemini Enterprise | **Custom MCP Server** |
| Cursor etc. | **MCP server** |

See https://relex.legal/docs/connectors
