# Positioning — Relex × any agent

**Relex doesn't replace your AI. It helps you use it end-to-end.**

Lawyers already use Claude, ChatGPT, Grok, Gemini, and other agents. What stops
them from using those models for *real* client work is the layer around the
model: confidential client data, firm know-how, client communication, payments,
and finding the next client. Relex is that layer.

Relex helps you use your agent end-to-end by:

- **Protecting PII and know-how.** Client names, national IDs, and contacts are
  sealed client-side under a PII password. The server stores only ciphertext.
  Document content is read and redacted by the Relex Private Model on Relex servers by default. The agent operates on
  structure and drafting and never receives plaintext PII.
- **Automating customer service.** Guest links, support agents, case agents.
- **Handling payments for free.** Billing stays in the browser; no card data in chat.
- **Opening a new market.** Be reachable through Relex.

The integration is PII-safe by construction: cryptographic sealing of parties,
redaction of documents, and an MCP `execute` guard that refuses plaintext PII
and returns deep links instead.
