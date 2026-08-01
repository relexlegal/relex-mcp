---
description: Connect this the agent to your Relex account (browser sign-in — no key paste).
---

# /relex-connect

Connect the agent to Relex. The default is a **browser sign-in over OAuth** — there
is no key to paste.

1. **Trigger the sign-in.** Make any Relex tool call (the simplest is the
   onboarding status read):

   ```
   execute({ method: "GET", path: "/onboarding/status" })
   ```

   If you're not connected yet, the Relex MCP server (`https://relex.legal/api/mcp`)
   replies with an OAuth challenge and your client opens the user's browser. Tell
   the user: "A browser window will open — sign in to Relex with Google or Apple
   and approve access, then come back." Wait for them to finish.

2. **Confirm.** Re-run the call (or `/mcp`) — `relex` should be connected with the
   tools `search` and `execute`, and the status call should return JSON. Say:
   "✅ Connected to Relex." Then offer: "Want me to set up your practice
   workflow?" → run `/relex-setup`.

   In the agent Desktop / your MCP client, add the connector under **Settings → Connectors
   → Add custom connector** with URL `https://relex.legal/api/mcp` (OAuth sign-in is
   automatic) — see `docs/connect-the agent-desktop.md`.

## Fallback — connect with a key (CI / headless / no-OAuth clients)

If a browser sign-in isn't possible, use a static API key instead:

1. In Relex: **Settings → API Keys → Create key**, copy it (shown once).
2. Add the server with the key as a bearer token:

   ```bash
   the agent mcp add --transport http relex https://relex.legal/api/mcp \
     --header "Authorization: Bearer rlx_..."
   ```

   (your MCP client: set the MCP server URL and `bearer_token_env_var = "RELEX_API_KEY"` —
   see `docs/connect-codex.md`.)

If `relex` does not connect, confirm the URL is reachable and, for the key path,
that the key is valid and active in **Settings → API Keys**.
