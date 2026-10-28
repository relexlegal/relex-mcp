---
name: relex-intake
description: Run an end-to-end client intake in Relex — find the request, write the intake note, make an agreement from a template, get it e-signed, and invoice the client. PII-safe id-only flow (client is a server-resolved guest ref, never a name/email). Organisation-only; notarization/e-filing are Enterprise.
---

# End-to-end client intake

You can take a matter from a client's first request all the way to a signed
engagement + an issued invoice, entirely over the MCP — without ever handling the
client's name, email, or ID. Identities are resolved server-side from ID-ONLY
references; you work in labels and refs.

```
find the request → snapshot + conflicts → write the intake → agreement from a
template → e-sign (id-only) → invoice → record
```

Everything here is **organisation-only** (agreements, invoices, clients are org
features). In a personal workspace these calls return `organization_required` —
tell the user to switch to (or set up) an organisation.

## 1 · Find the request

Two sources, both PII-safe:

- **By email** — mail reaches Relex through a mailbox connector the user
  connects in the browser (`https://relex.legal/settings/connectors`); there is
  no sync endpoint over MCP. Ingested mail lands in private knowledge, redacted
  on ingest. `execute GET /knowledge` lists sources as METADATA ONLY (filename,
  folder, status, page/party counts, dates) — never content. To read the
  substance of a request, run a case turn (`POST /agent {type:"case_req"}`,
  which searches the redacted corpus) or hand the user the knowledge deep link.
- **By guest link** — the client joined a case via a guest link. Read the roster
  with `GET /ontology/case/{caseId}/participants` (`relex-participants`): the
  client is usually `[GUEST_1]`. Read what they wrote via
  `execute GET /cases?caseId={caseId}&full=true` (note: caseId is a QUERY param;
  `full=true` is REQUIRED here — a plain `GET /cases` returns a bounded summary
  without the timeline phases) — their request lives in the case `timeline`
  phases + `notes`.

## 2 · Snapshot + conflicts

Before drafting, run the 5-sentence snapshot and the conflict check — that
discipline lives in `relex-counsel` and `relex-matter` ("Intake & conflicts").
Lock who the client is (as a label), the forum, and the deadline into the case.

## 3 · Write the intake note

`execute POST /cases/{caseId}/attachments` `{ attachmentName, content }` — a
label-only markdown note recording the request, scope, and your intake analysis.
De-identified: parties as `[PARTY_NAME_n]`, never a real name or contact. This is
the correct "record to the case" path.

## 4 · Agreement from a template

- `execute GET /templates` (`?organizationId=` for the org) — the practice's
  reusable engagement/agreement templates (generic role placeholders, no PII).
- `execute POST /templates/{templateId}/instantiate` `{ caseIds: [caseId] }` —
  creates a draft agreement from the template and links the case (so a guest on
  that case can be a signer). Returns the new agreement.
- If no suitable template exists, hand the user the dashboard to create one (a
  template body is authored in the browser, not by you — keep model-authored HTML
  off the signing path).

## 5 · E-sign — id-only signers

`execute POST /agreements/{agreementId}/send` with **`signerRefs` only** — never a
name, email, or `documentHtml`. Each ref carries params from `search`
(`guestUserId` **or** `memberSelf`, `role`, `routingOrder`) — e.g. the client as
`{guestUserId, role:"client", routingOrder:1}` then you as
`{memberSelf:true, role:"firm", routingOrder:2}`.

The server resolves each ref to a name+email server-side (from the case's guests /
your account), substitutes signer names into the STORED template body
(`[SIGNER_1_NAME]`, `[<ROLE>_NAME]`), and emails each signer a secure no-login
link in routing order — a signed PDF + Certificate of Completion (ESIGN/UETA &
eIDAS). Poll `GET /agreements/{id}` until `status: "executed"`.

- The client's guest userId comes from the participants roster (their ref), not
  from you typing anything.
- A **sealed client party** (not a guest) can't be signed over the API — its
  identity is encrypted. Send the user the case deep link to sign in the browser.
- Sending `documentHtml` or raw `signers` is **refused** — that is the correct
  guard, not an error. Use `signerRefs`.
- `memberSelf` resolves to the *calling account*. In a shared channel (the agent in
  Slack) that is the connecting admin, which may not be the matter's lawyer —
  confirm who the firm signer should be before sending.

## 6 · Invoice the client

`execute POST /invoices` `{ caseId, clientRef: { guestUserId }, items: [{
description, qty, unitAmount }] }` — amounts are the firm's own figures in minor
units (e.g. cents). Then `execute POST /invoices/{invoiceId}/send`:

- If the organisation is registered in the **partner program** (which enables
  Stripe), the response carries a `paymentUrl` — a card-payable hosted invoice,
  emailed to the client.
- If not, the response carries `needsPartnerRegistration` + a `registerUrl` —
  hand the user that link to register, or they can collect out of band and you
  `execute POST /invoices/{invoiceId}/mark-paid`.

The client's email is resolved server-side from the guest ref — never quote or
handle it. Never quote Relex's own plan prices; invoice amounts are the firm's.

## 7 · Entitlements — surface, don't fight

- **Agreements, invoices, clients = organisation** workspaces. `organization_required`
  → the user must switch to / create an org.
- **Notarization + e-filing = Enterprise**, and both end in a HUMAN step (you can
  prepare/search e-filing and check notarization capability, but a person
  executes the notary session / court filing). A `requires_enterprise_plan` reply
  → explain + point to billing; don't retry.
- On any 402/403, relay the deep link and move on — that is the path.

## Anti-patterns

- Typing or pasting a client's name/email into a send or invoice body. Use refs.
- Sending `documentHtml`/`signers` over MCP (refused). Instantiate a template.
- Treating a sealed client like a guest for signing — it must go through the
  browser.
- Quoting Relex plan prices, or retrying a `requires_enterprise_plan` /
  `organization_required` refusal instead of relaying the link.
- Running intake in a personal workspace — agreements/invoices are org-only.

## Alongside

- `relex` — connect, the connector tools and workflow, the one PII rule.
- `relex-participants` — the roster; the client is usually `[GUEST_1]`.
- `relex-matter` / `relex-counsel` — the intake snapshot, conflicts, deadlines.
