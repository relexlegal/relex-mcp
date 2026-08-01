# Install Relex via generic MCP

One remote MCP server powers every client:

```
https://relex.legal/api/mcp
```

You do **not** run a local server. Point your MCP host at that URL, sign in
once in the browser, and your agent can operate on Relex cases without ever
receiving client PII.

## 1. Choose your path

| You use… | Package | Start here |
|----------|---------|------------|
| Claude Code / Claude desktop / claude.ai | [relex-claude](https://github.com/relexyou/relex-claude) | Plugin marketplace or custom connector |
| ChatGPT / Codex | [relex-gpt](https://github.com/relexyou/relex-gpt) | Custom connector / MCP app |
| Grok (xAI API or Grok Build) | [relex-grok](https://github.com/relexyou/relex-grok) | Remote MCP tool |
| Gemini CLI / Gemini Enterprise | [relex-gemini](https://github.com/relexyou/relex-gemini) | `gemini mcp add` or Enterprise data store |
| Cursor, Windsurf, Cline, Continue, custom | **This repo** | Config below |

## 2. Personal plan vs Team plan

### Personal (Pro / Max / Plus / individual seats)

1. Open your client's **Settings → Connectors / MCP / Tools**.
2. **Add custom connector** / **Add MCP server**.
3. URL: `https://relex.legal/api/mcp`
4. Save, then click **Connect** (or run the first tool call).
5. Browser opens → sign in to Relex with Google or Apple → approve.
6. Status shows **Connected** with tools `search` and `execute`.

### Team / Business / Enterprise

| Role | What you do |
|------|-------------|
| **Owner / admin** | Install the connector **once** in the organisation's admin settings (Connectors, Apps, or Plugins). Use URL `https://relex.legal/api/mcp`. Optionally publish it to the workspace so all members see it. |
| **Member** | You **cannot** add custom connectors yourself. Open **Settings → Connectors** (or Apps). Find **Relex** under available / workspace connectors. Click **Connect** and complete OAuth for **your** Relex account. |

If members don't see Relex:

- Admin has not added or published the connector yet → ask admin.
- Admin added it but you haven't connected → click **Connect** (this is normal).
- Wrong workspace or role → check with admin that your group is allowed to use custom connectors / developer mode.

**Important:** Admin install ≠ member auth. Each person still signs into their own
Relex account. The admin only makes the connector *available*.

## 3. Generic JSON config

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

Variants some clients expect:

```json
{
  "mcpServers": {
    "relex": {
      "url": "https://relex.legal/api/mcp"
    }
  }
}
```

```json
{
  "mcpServers": {
    "relex": {
      "httpUrl": "https://relex.legal/api/mcp"
    }
  }
}
```

## 4. After connect — practice workflow

Say to your agent:

> Set up my practice workflow with Relex

It will:

1. Confirm OAuth (browser if not already done).
2. Deep-link you to set a **PII password** (encrypts client identities in-browser).
3. Guide you to upload **know-how**.
4. Auto-create **encrypted parties** (agent only sees counts, never names).
5. Offer to start the first case.

## 5. API-key fallback (CI / headless)

1. Relex → **Settings → API Keys → Create key** (shown once).
2. Pass as bearer token:

```bash
# Example — Claude Code
claude mcp add --transport http relex https://relex.legal/api/mcp \
  --header "Authorization: Bearer rlx_..."
```

```json
{
  "mcpServers": {
    "relex": {
      "httpUrl": "https://relex.legal/api/mcp",
      "headers": {
        "Authorization": "Bearer rlx_..."
      }
    }
  }
}
```

Revoke anytime under **Settings → API Keys** or unpair clients under
**Settings → Agents**.

## 6. Security summary

- Client PII is sealed client-side under your PII password; the server stores ciphertext only.
- `execute` refuses party/document plaintext and returns deep links instead.
- The agent works with labels like `[Party 1]` and anonymized counts.
- See [SECURITY.md](../SECURITY.md).
