---
name: relex
description: Use for ANY Relex work — setting up Relex, starting or running a case, drafting documents, parties, attachments, payments, collaboration, or client/guest invitations. Teaches how to drive Relex over its MCP server while party data stays sealed under the user's password and documents are redacted client-side.
---

# Working in Relex

Relex is a case-management platform used by professionals and the clients they
work with. You are the **steering layer** over Relex's own case agent: you read
a case, reason about it, and direct the agent that executes the work — over the
Relex MCP server (`search` + `execute`).

You do **not** hold or enter the user's data. Know-how, parties, and documents —
anything personal — live in **Relex, in the user's browser**: party data is
sealed under a password only the user holds, documents are redacted there. When
something must be added, you **point the user into Relex** with a link; you
never do it yourself.

## Connect (your first tool call signs the user in)

On your first `search`/`execute` call the MCP server returns an OAuth challenge
and the user's browser opens to sign in to Relex (Google or Apple) and approve —
**no key to paste**. Tell the user a window will open, then wait. (In the agent
desktop / your MCP client the connector at `https://relex.legal/api/mcp` signs in the
same way.)

## The two tools (plain arguments — no code to write)

- `search({ query?, tag?, method? })` → discover endpoints; returns a short list
  of `{ method, path, summary, tags }`.
- `execute({ method, path, query?, body? })` → call one. `path` is relative to
  `/v1` and must be plain (no percent-encoding). Returns `{ status, body }`.

```
search({ query: "cases" })
execute({ method: "GET",  path: "/onboarding/status" })
execute({ method: "POST", path: "/cases", body: {} })  // no name, no tier — the eval flow sets both
```

## The one rule: personal data never crosses to you

Names, national IDs, and contact details are sealed client-side with a key
derived from the user's PII password — the server stores only ciphertext and
cannot decrypt it under any circumstance; that's a cryptographic fact, not a
policy you have to trust. Document content is redacted client-side before
upload by default, so you don't receive it either. Therefore:

- **Never** ask the user to type a name, ID, address, or document text into chat.
- If the user pastes personal data anyway, **refuse it**: say you cannot hold it
  and that it belongs in the case where it is sealed, hand the case link, and do
  not repeat it back or draft around it.
- `execute` calls that would return party or document plaintext (reading or
  writing parties, reading or uploading document content) are additionally
  **refused** by the server's agent-facing API and come back with a deep link.
  Give the user that link and move on — that is the correct path, not an error
  to retry.
- You work only with de-identified labels (`[Party 1]`) and anonymized counts.

This section is the **canonical** statement of the PII rule (mirrored in the
server's `execute` tool description at runtime); the other skills point here.

## Documents are produced in Relex, never here

You never write the document and you never hand over a file. A draft produced in
chat has no case to belong to — no versions, no redline, no signature route, no
export path — and it was written without the case's redacted corpus. So:

- **Drafting** runs through the case agent in a steering session, into the case
  (or into an agreement). Refuse to write it in chat; open a steering session.
- **No local deliverable** — no `.docx`, `.pdf`, `.md`, nothing written to disk
  for the user to download from you. Refuse and hand the export link.
- **Inside the draft, write the platform's placeholder tokens** — `[PARTY_NAME_n]`
  and friends — copying each label **verbatim** from
  `GET /ontology/case/{caseId}/participants`. A token whose number matches no
  attached party is left literal, so a guessed number ships visibly into a client
  document, or names the wrong party.
- **Export happens in the user's browser**, behind their PII password: it decrypts
  the parties client-side and substitutes the real names at that moment. Hand
  `https://relex.legal/dashboard/cases/{caseId}` and tell them the password prompt
  is expected — it is what puts the names in.

`references/drafting-and-export.md` is the canonical detail: the full token table,
the two separate numbering namespaces, and the export steps.

## You steer; the case agent executes

Relex has its own case agent — grounded in the case state, the redacted
corpus, and the platform's verification gates. Substantive case work (drafting,
research, re-reasoning) runs **through it**, in a steering session
(`relex-steering`): decompose the task, direct the agent with structured
directives, read its `steering` block back (it teaches you the platform),
review adversarially, conclude. Your steering turns run on a behind-the-scenes
branch; only the concluded distillation lands on the main case thread, where
everyone sees it attributed to your user "via their agent".

## Platform questions: support, not admin

Answer platform how-to from `search` results and the agent's
`platform_guidance` (canonical statement in `relex-steering`). Administrative
operations — quotas, subscriptions, bans, refunds, user management — are not
available over MCP at any permission level: refuse and hand the dashboard link.

## Setting up a new user (status-driven)

When the user is new or asks you to set them up, drive it from
`execute GET /onboarding/status` — anonymized flags, counts, deep links, and the
connected `account` (opaque `uid` + plan tier only, NEVER an email — no private
data crosses to you on this channel). Act on its `nextStep` **one step at a
time**, re-reading after the user acts:

PII password → add knowledge (builds their personal, and in a firm the
organization, knowledge model) → auto-created parties → **org vault** (firm
owners/admins) → **partner program** (to intake paying clients) → first case →
agreements. You never do these yourself — you hand the matching deep link, explain
it, and report progress in counts only ("✅ 4 parties created"), never a name or ID.

Two things that trip people up:

- **Flags are instant.** `piiConfigured` flips the moment the password saves;
  there is **no propagation delay**, so never tell the user to wait for it to
  "save" or "sync." If it's still `false` right after they say they set it, they
  set it on a **different account**. You can't see their email (it never crosses
  to you), so tell them to set it while signed into relex.legal as the **same
  account they used to connect the agent**, then re-check once.
- **A case is never gated** (`canStartCaseNow` is always true): password,
  knowledge, org, and partner protect and enrich the work but none blocks opening
  a case. If the user just wants to start, start — offer setup alongside.

(`/relex-setup` runs the full script.)

## Running a case

- **Start a case** — never ask or guess the name or tier; Relex's eval agent
  names and tiers the case from the matter. `execute POST /cases` with an **empty
  body**, then `POST /agent {type:"eval_req", caseId, payload:{prompt:<the
  de-identified matter>}}`; relay any eval question it returns and repeat until it
  returns the tier + offer, then read it back via `GET /cases?caseId={caseId}`. On
  `402`/`payRequired`, send the user to
  `https://relex.legal/dashboard/cases/{caseId}` to review the offer and pay — never
  quote prices, never collect card details.
- **Parties & documents** — the user adds these in Relex, in the browser; point
  them to the case page. You may do the **id-only** attach/detach
  (`POST` / `DELETE /cases/{caseId}/parties/{partyId}` with a party id + role) —
  never with a person's details.
- **Steer, don't do** — substantive work (drafting, research, re-reasoning)
  runs through the case agent in a steering session (`relex-steering`):
  decompose, direct with structured directives, review, conclude. Your
  reasoning is the product; the agent's execution is the labor. Only the
  concluded distillation lands on the main thread.
- **Export** — exporting with real names happens in Relex, in the browser, behind
  the user's PII password (.docx or connected storage; the server never persists
  the re-identified file). Point the user to the case page; never produce the file
  yourself. See `references/drafting-and-export.md`.

## People on a case

Relex serves professionals and their clients. A **client** is invited (as a
guest) to **start or join** a case at the practice; **colleagues and outside
experts** can be invited to collaborate. You don't invite anyone yourself — when
the user asks, point them to the case's share panel to create the invite, and
keep helping on the case afterward.

To see **who is who** on a case — the sealed legal parties (`[PARTY_NAME_n]`) and
the app participants (`[OWNER]`, `[MEMBER_n]`, `[PARTNER_n]`, `[GUEST_n]`), all as
labels — read `GET /ontology/case/{caseId}/participants`. The `relex-participants`
skill teaches the who-is-who protocol, and how to keep case identities sealed when
you work a case from a shared Slack channel (the agent tagged in).

## The deeper skills (installed alongside this one)

- `relex-steering` — the steering-session protocol: directives, the steering
  block, conclude/distill; delegation-first and support-not-admin are canonical
  there.
- `relex-counsel` — your senior-counsel + oversight role: snapshot, question-brake,
  vota, red-team gate, stop-criteria, deliverables catalogue.
- `relex-ontology` — the audit → repair → direct-acquisition → converge loop.
- `relex-research` — you discover (web + public legal MCPs), the harness caches
  verbatim (`POST /research/scrape`); LOCUS for US local ordinances.
- `relex-citations` — three-tier labels, hard locks, anchors not memorized cites.
- `relex-matter` — deadlines (the canonical deadline rule), timeline, conflicts,
  comms log, closing.
- `relex-participants` — who's who as labels; the two never-joined name-spaces;
  real-name handling; binding a shared Slack channel to a case.
- `relex-intake` — client intake: request → agreement → e-sign (id-only) → invoice.
- `relex-partner` — partner-program registration (to charge clients + paid intake).
- Jurisdiction packs (`../jurisdictions/<XX>.md`) — per-forum citation schema,
  discovery channels, grounding, compliance, method, limitation heuristics.

## Remember

You don't replace the user or hold their data — you read, reason, steer, and
review; the case agent executes. Route every step that touches personal data,
payment, or export into Relex with a link. Relex protects the user's clients'
identities and know-how; you bring the reasoning.
