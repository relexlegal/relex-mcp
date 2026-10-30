# How Relex works (load on demand)

The whole platform on one page for an assistant connected to the Relex MCP server
(`https://relex.legal/api/mcp`, thirteen tools). It describes what each tool does
and how data moves. The details of drafting and export are in
`drafting-and-export.md`; ontology work is in the `relex-ontology` skill.

## 0 · Relex at a glance

Relex has **three products**:

| Product | What it is | Over this connector |
|---|---|---|
| **Legal Workspace** | The user's own workspace: matters, practice know-how, ontology and the work saved by every connected assistant | Yes: the matter tools plus `search` / `read_api` / `execute` |
| **Legal SDK** | A separate library for building a firm's or legal department's own platform | No: it is not served by the MCP server |
| **Legal Marketplace** | Published, verified profiles of legal professionals | Yes: `find_legal_professionals`, `read_legal_professional`, `prepare_professional_request` |

The workspace is **portable**: what is saved in Relex (matters, work products,
steering conclusions, the matter and firm ontologies, know-how) is reachable from
any authorised MCP client: Claude, ChatGPT, Grok, Gemini, Perplexity or another
host. Portability covers what is saved in Relex, not the import of chat
histories from other assistants.

Each workspace area is a `search` tag, so `search({ query, tag: "Know-how" })`
lists only that area's endpoints. The full table is in §7.

| Area (`tag`) | What the assistant can do |
|---|---|
| Matters | List, read and work on matters: context, sources, parties as labels, actions, timeline, attachments |
| Portable context | Read a matter's context and earlier work; conclude a steering session onto the matter |
| Ontology | Read and correct the matter graph; read the firm ontology |
| Know-how | Playbooks, the clause library, templates, knowledge sources, firm ontology |
| Research | Cache official legal text (`POST /research/scrape`); US local ordinances |
| Data collection | See which channels feed the workspace (status only; connecting is done in Relex) |
| Agreements | Drafts, review comments, enforceability advisory, agreements from templates |
| Intake | Evaluation agent (`POST /agent`), guest-link redemption |
| E-signature | Send an agreement for signature; read signature status by signer role |
| Invoices | Create, read and send organisation invoices |
| Payments | Read payment status on a matter and online-payment readiness |
| Marketplace | Find and read professionals; prepare a request the user approves |
| Account | Onboarding status, profile, organisations |
| eFiling | US court e-filing capability, search and preparation |

---

## 1 · The rule: matter data enters and leaves through Relex

```
 user's files ──upload on the case page──▶ Relex ──redacted text──▶ assistant
                                            ▲                          │
                 export in the Relex app ◀──┘◀── save_matter_work_product (labels)
        (labels → real details, in the user's browser)
```

| Direction | How | Where |
|---|---|---|
| **In: documents** | The user adds files with **Attach** (document, folder, photos, audio) on the case page. Relex reads and redacts them. | `https://relex.legal/dashboard/cases/{caseId}`, also returned as `deepLink` / `upload.deepLink` |
| **In: what the assistant reads** | `read_matter_context` (parties as labels, documents, readiness, earlier work), `read_matter_source` (full redacted text, paged) | MCP |
| **Out: what the assistant writes** | `save_matter_work_product`: markdown with **label tokens**, saved as a versioned draft in the matter | MCP |
| **Out: the finished document** | The user exports from the case page (Word, PDF, Google Drive, signing, email/print). The browser swaps the labels for the parties' real details at that moment, after the user's PII password when parties are sealed. | Relex app |

What follows from this:

- **Missing documents** → the user uploads them on the case page (hand the
  `upload.deepLink` from `read_matter_context` / `diagnose_matter_sources`). Raw
  documents, names, IDs or addresses pasted into chat are outside the seal; the
  case page is where they belong. `execute` refuses `POST /cases/{caseId}/files`
  for the same reason and returns the case page.
- **A finished document** → saved with `save_matter_work_product`, then exported
  by the user from Relex. The assistant produces no `.docx`/`.pdf` of its own:
  only Relex can put the real names in, and only in the user's browser.
- **Original files and identity fields** never reach the assistant. Every tool
  returns redacted text and labels; `read_api` on an original file or the
  identifying roster returns a pointer to the browser instead.

## 2 · Labels: the vocabulary of every draft

Parties are sealed under the user's password. The assistant sees a label for
each, from `read_matter_context` → `parties[].label` (or
`read_api /ontology/case/{caseId}/participants`):

```json
{ "label": "[PARTY_NAME_2]", "role": "respondent", "description": "…" }
```

| Token | Replaced at export with |
|---|---|
| `[PARTY_NAME_n]` | the party's name |
| `[PARTY_ADDRESS_n]` | address |
| `[NATIONAL_ID_PLACEHOLDER_n]` | personal ID (ID card, passport, CNP, NI number…) |
| `[COMPANY_REG_PLACEHOLDER_n]` | company registration number |
| `[TAX_ID_PLACEHOLDER_n]` | tax / VAT number |
| `[BANK_ACCOUNT_PLACEHOLDER_n]` | IBAN / account |
| `[PHONE_PLACEHOLDER_n]` | phone |
| `[EMAIL_PLACEHOLDER_n]` | email |

- `n` is the party's slot, copied from its label: every token for the same party
  carries the same `n`. "Party 2" in the app is only a display name; the token is
  `[PARTY_NAME_2]`.
- Uploaded documents can also show the short forms `[NATIONAL_ID_1]`,
  `[TAX_ID_1]`, `[BANK_ACCOUNT_1]`, `[PHONE_1]`, `[EMAIL_1]`; export resolves both.
- A token whose `n` matches no party, or a detail the party record does not hold,
  stays in the exported document as written.
- `[DATE_PLACEHOLDER]`, `[AMOUNT_PLACEHOLDER]` and typed placeholders such as
  `[amount in EUR]` are **not** filled at export: they mark facts for the human.
- `save_matter_work_product` scans content for raw personal data. A real name,
  ID, address, IBAN, phone or email returns `outcome: "blocked"` with
  `code: "pii_in_content"`, the categories found, and a repair hint naming the
  labels. Replace and save again.

## 3 · The tools

| Tool | What it does | Notes |
|---|---|---|
| `list_matters` | Up to 20 matters: `caseId`, `deepLink` | No names or client identity |
| `read_matter_context` | Parties (labels), participants, documents (redacted text or outcome + repair link), `sourceReadiness`, `workProducts` (draftId, revision), `steeringConclusions`, `upload` | Page with `sourceOffset = nextSourceOffset` while `sourceReadiness.hasMore` |
| `diagnose_matter_sources` | Per-source `outcome`, `failureCode`, `failureMessage`, `repairAction`, `deepLink`; `upload` | `repairAction`: `raise_quota`, `retry_processing`, `reprocess_or_replace_source`, `open_in_relex` (done in Relex), `wait_for_processing`, `read_matter_source` (continue over MCP) |
| `read_matter_source` | Full redacted text of one source, in pages from `offset` | Next page: `textOffset + len(text)` until `hasMoreText=false` |
| `save_matter_work_product` | `{caseId, draftName, content, draftId?, expectedRevision?, idempotencyKey?, evidenceRefs?}`: saves markdown as a draft; a new version with `draftId` + `expectedRevision` | Returns `outcome, draftId, revision, readback, deepLink, export`; `blocked` (ungrounded citations or personal data, with repair), `conflict` (stale or missing `expectedRevision`); repeating `idempotencyKey` returns the original |
| `correct_matter_ontology` | Typed operations on the matter graph at `baseRevision` | See §4 |
| `conclude_matter_session` | Ends a steering session this client opened on `POST /agent` (`case_req`); conclusions land in the matter | 404 when no open session |
| `find_legal_professionals` / `read_legal_professional` | Published, verified Marketplace profiles | Office ≠ admission jurisdiction |
| `prepare_professional_request` | A request linking a matter and a professional; returns an approval link | Nothing is sent, shared or charged until the user approves in Relex |
| `search` | Finds `/v1` endpoints by query, tag or method | |
| `read_api` | One GET on `/v1` | Read-only |
| `execute` | One POST/PUT/PATCH on `/v1` | Deletions and irreversible routes are done in the Relex dashboard |

Results are minimised: only the fields an assistant needs come back, and any
`hint` / `upload` / `export` field is a plain pointer.

## 4 · The ontology (the matter's knowledge graph)

Each matter has a graph of parties, issues, obligations, deadlines and facts,
built from its sources.

- **Read:** `read_api /ontology/case/{caseId}` → digest with `revision`, issues,
  deadlines and conflicts; `?view=full` returns the whole graph. Labels only.
- **Correct:** `correct_matter_ontology { caseId, baseRevision, idempotencyKey,
  operations, evidenceRefs, rationale? }` with `baseRevision` = that `revision`
  (up to 50 operations and 50 evidence refs).

| `op` | Fields |
|---|---|
| `upsert_entity` | `id, type, label, properties` |
| `update_entity` | `id, type?, label?, properties?` |
| `merge_entities` | `from_id, into_id` |
| `upsert_relationship` | `source_id, verb, target_id`; optional `description`, `confidence` (default 0.8) |
| `set_issue_status` | `id, status` (`open`, `contested`, `settled`) |
| `delete_entity`, `delete_relationship` | Not applied over MCP: a batch containing one is refused whole with `outcome: "browser_required"` and `removalOperationIndexes`; the user confirms removals in Relex at `deepLink`, and the remaining operations can be sent again without them |

- `evidenceRefs: [{ sourceId, provenanceClass }]`, where `provenanceClass` is
  `uploaded_evidence` (default), `official_authority` or `user_confirmation`.
  `agent_work_product` cannot corroborate a claim and is rejected.
- Outcomes: `ready` with `revision` + `results[{op, ok, detail}]`;
  `conflict` / `revision_mismatch` when the graph moved on (re-read, re-apply);
  `grounding_blocked` when a claim has no supporting source.
- `POST /ontology/case/{caseId}/reconcile` rebuilds the deterministic backbone
  from case data; `POST /ontology/case/{caseId}/agent` runs the editor agent from
  a natural-language instruction.

## 5 · Agents and sessions

- **Intake (eval):** `execute POST /cases {}` creates an empty matter; then
  `execute POST /agent {type:"eval_req", caseId, payload:{prompt}}` with
  de-identified facts. The reply is either questions or an evaluation with a
  tier. HTTP 402 means a plan is needed; the case page shows the options.
- **Case agent (steering):** `POST /agent {type:"case_req", caseId,
  payload:{prompt}}` runs one turn of the Relex case agent on a steering branch.
  `conclude_matter_session` saves the distilled conclusions to the matter, where
  every connected host sees them in `steeringConclusions`.
- **Continuity across hosts:** `workProducts` and `steeringConclusions` in
  `read_matter_context` hold what earlier sessions (Claude, ChatGPT, Grok,
  Gemini, Perplexity, any MCP client, or the Relex app) saved. Continue from them.

## 6 · Grounding

- `POST /research/scrape` fetches and caches the verbatim text of an official
  legal source; `GET /research/sources` lists what can be cached.
  `GET /research/locus` covers US local ordinances.
- A citation is grounded when it comes from cached official text or a ready
  source. `save_matter_work_product` blocks ungrounded citations; `[UNVERIFIED]`
  in the same sentence marks one the user has to check.

## 7 · Every workspace area (via `search` / `read_api` / `execute`)

| Area (`tag`) | Endpoints |
|---|---|
| Matters | `GET /cases`, `POST /cases`, `GET /cases/{caseId}/draft`, `POST /cases/{caseId}/attachments` (markdown notes, labels only), `GET /actions/pending`, `GET /cases/{caseId}/actions`, `POST /cases/{caseId}/actions/{actionId}/complete`, timeline |
| Portable context | `GET /cases/{caseId}/context`, `POST /cases/{caseId}/steering/conclude`, `GET /ontology/case/{caseId}`, `GET /ontology/firm` |
| Ontology | `GET /ontology/case/{caseId}`, `POST /cases/{caseId}/ontology/corrections`, `POST /ontology/case/{caseId}/reconcile`, `POST /ontology/case/{caseId}/agent`, firm ontology |
| Know-how | `GET /playbooks`, `POST /playbooks`, `GET /clauses`, `POST /clauses` (`scope=user` or `org`), `GET /templates`, `GET /knowledge`, `POST /knowledge/{sourceId}/retry` |
| Research | `POST /research/scrape`, `GET /research/scrape/{jobId}`, `GET /research/sources`, `GET /research/locus` |
| Data collection | `GET /users/me/connectors`, `GET /organizations/{organizationId}/connectors`: provider, kind and status; mailbox addresses and credentials stay in Relex |
| Agreements | `GET /draft/{draftId}`, `GET` / `POST /cases/{caseId}/draft/{draftId}/comments` (`visibility`: `internal` or `shared`), `POST /cases/{caseId}/draft/{draftId}/enforceability-advisory`, `POST /templates/{templateId}/instantiate`, `POST /agreements` |
| Intake | `POST /agent` (`eval_req`, `case_req`), `POST /guest-links/{token}/redeem` |
| E-signature | `POST /agreements/{agreementId}/send` (id-only signers), `GET /cases/{caseId}/draft/{draftId}/execution` (status per signer role), `GET /notarizations/capability` |
| Invoices | `GET /invoices`, `GET /invoices/{invoiceId}`, `POST /invoices`, `POST /invoices/{invoiceId}/send`, `POST /invoices/{invoiceId}/void`; marking paid is done in Relex |
| Payments | `GET /cases/{caseId}/partnerPayments` (status, amount, currency), `GET /partners/connect/status` (readiness flags) |
| Account | `GET /onboarding/status` (flags with `deepLink`s), `GET /users/me`, organisations |
| eFiling (US) | `GET /efilings/capability`, `POST /efilings/search`, `POST /efilings/prepare` |
| Decisions | `POST /jev/decide` (when enabled) |

Text written into know-how and comments (`POST /clauses`, `POST /playbooks`,
draft comments) is checked for personal data like a saved work product: write
label tokens, not real identifiers. A comment's author is the connected Relex
account.

Not over MCP: deletions, billing and payment writes (charges, refunds, payouts,
marking an invoice paid), uploads, connecting channels, quotas and admin; those
live in the Relex app.

## 8 · Reading modes and limits

- By default documents are read by the **Relex Private Model on Relex servers**.
  Browser mode reads on the device. Private-server and custom modes send raw
  pages to the reader the user chose.
- Redaction is automated and can miss identifiers; the assistant does not promise
  universal anonymity.
- Only `relex.legal` is the Relex service.

## 9 · A complete pass, end to end

1. `list_matters` → `caseId`.
2. `read_matter_context` → parties (labels), documents, earlier work.
3. `diagnose_matter_sources` → anything blocked or missing goes back to the user
   as the case-page link (upload, retry, replace).
4. `read_matter_source` for long sources; `POST /research/scrape` for the law.
5. Optionally `read_api /ontology/case/{caseId}` and `correct_matter_ontology`
   with evidence.
6. Write the document with labels → `save_matter_work_product` (read back the
   `readback`).
7. Hand the user the `export.deepLink`: they open the draft in Relex and export
   it to Word, PDF, Drive, signing or email; the password prompt is what puts the
   real names in.
