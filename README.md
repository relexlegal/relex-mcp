# Relex × MCP (generic)

**Legal Workspace — One source of truth for any Agent — confidential by design**

Keep legal knowledge, matter context and saved progress in Relex, independently
of the assistant you use. Authorize another compatible agent to continue from
the same stored context, without rebuilding the background in another chat.

## Portable context, with your permission

1. Create or open the matter in Relex and add information through its protected
   intake and document workflows.
2. Connect a supported client to `https://relex.legal/api/mcp` and authorize
   your own Relex account. Installing a package does not authorize private data.
3. Ask the agent to read the permitted matter context before working and save
   its conclusions when finished. A second authorized client can then use that
   continuing record.

Portability covers information saved in Relex, not automatic import of private
chat histories or a model's internal memory. Client-side identity encryption,
de-identification and MCP access controls protect the supported workflows;
de-identified legal facts may still be sensitive. Review what you authorize.

## Workspace, SDK and Marketplace

Use **Legal Workspace** for persistent legal context; the **Legal SDK** for
building your firm's or legal department's own platform; and the
**Legal Marketplace** to discover published professional profiles or make an
AI-first law firm discoverable across specialties.

An agent may help find a professional and prepare a reference-only request.
The user must review and approve sharing in Relex. Discovery is not engagement,
a completed conflict check, payment or a guarantee of professional availability.

Client support depends on the host product, plan and administrator settings.
Gemini CLI support does not imply support in every Gemini web experience.
Harvey BYOMCP is a customer-admin connection path, not a claim of Harvey
Connector Library listing or approval. Check the current
[connector guides](https://relex.legal/docs/connectors) and
[portable-context guide](https://relex.legal/guides/portable-legal-context).
## MCP endpoint (the whole product surface)

```
https://relex.legal/api/mcp
```

Transport: **Streamable HTTP** (MCP). Auth: **OAuth 2.1 + PKCE** (browser
sign-in with Google/Apple — **no key to paste**). Static API keys work as a
CI/headless fallback.

Two tools, fixed ~1k-token footprint:

| Tool | Purpose |
|------|---------|
| `search` | Discover API endpoints from the OpenAPI surface |
| `execute` | Call one validated endpoint with the user's auth + PII guard |

The MCP handler lives in the **Relex backend** (`/v1/mcp`). This repo is
configuration, skills, and install docs — not a separate server to host.

## Quick connect (any MCP client)

### JSON config (Cursor, Windsurf, Claude Desktop, etc.)

```json
{
  "mcpServers": {
    "relex": {
      "type": "http",
      "url": "https://relex.legal/api/mcp"
    }
  }
}
```

Some clients use `url` / `httpUrl` instead of `type`+`url`. Same endpoint.

### CLI examples

```bash
# Claude Code (HTTP + OAuth)
claude mcp add --transport http relex https://relex.legal/api/mcp

# Gemini CLI
gemini mcp add --transport http relex https://relex.legal/api/mcp

# API-key fallback (CI / headless)
claude mcp add --transport http relex https://relex.legal/api/mcp \
  --header "Authorization: Bearer rlx_..."
```

After connect, say: **"Set up my practice workflow with Relex."**

## What each product calls this

Same MCP URL; different UI labels:

| Product | Official UI name |
|---------|------------------|
| Claude | **Custom connector** |
| ChatGPT | **App** / custom MCP connector (Developer mode) |
| Grok.com | **Connector** → Custom |
| Grok Build | **MCP server** / **plugin** |
| Gemini CLI | **MCP server** |
| Gemini Enterprise | **Custom MCP Server** |
| Cursor / generic | **MCP server** |

Always name it **Relex**, URL `https://relex.legal/api/mcp`.

## Personal vs Team (applies across hosts)

| Plan type | Who installs | Who signs in |
|-----------|--------------|--------------|
| **Personal** | **You** add the connector / app / MCP server | **You** Connect + OAuth |
| **Team / Business / Enterprise** | **Owner or admin** adds once | **Each member** finds Relex listed and **Connect**s *their* Relex account |

Members on Team plans usually **cannot** add custom entries themselves.

## OAuth (generic MCP)

Hosts that support MCP OAuth: unauthenticated call → `401` +
`WWW-Authenticate: resource_metadata=…` → browser Google/Apple on relex.legal →
bearer token on later `search`/`execute`. Full walkthrough:
[`docs/oauth.md`](docs/oauth.md) and https://relex.legal/docs/connectors/mcp

Full install flows: [`docs/install.md`](docs/install.md).

## Layout

```
relex-mcp/
├── plugin/
│   ├── .mcp.json              Remote MCP connector URL
│   ├── skills/                PII-safe workflow skills (agent-agnostic)
│   ├── agents/                Onboarding guide
│   ├── commands/              /relex-setup, /relex-connect
│   └── references/
├── docs/
│   ├── install.md             All clients + personal vs team
│   ├── connect-generic.md     Cursor, Windsurf, custom hosts
│   └── positioning.md
├── SECURITY.md
└── README.md
```

## What the agent can do

Act as **senior counsel + steering layer** over Relex's execution harness:
start cases, steer drafting sessions, audit the case ontology, ground citations,
run intake → e-sign → invoice — while **never** receiving names, national IDs,
or document plaintext. Those steps deep-link into the browser.

## Docs on relex.legal

- [MCP Server overview](https://relex.legal/docs/mcp)
- [Connectors hub](https://relex.legal/docs/connectors)
- [For AI Agents](https://relex.legal/for-agents)

## License

AGPL-3.0-or-later — see [LICENSE](LICENSE).
