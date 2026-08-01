# Drafting and export (load on demand)

Where documents are produced, what to write inside them, and how the user gets a
finished file with real names in it. This is the **canonical** statement of the
production rule; `relex-counsel` and `relex-steering` point here.

The short version: **the document is produced inside Relex, never in this chat.**
You direct the case agent to draft into the case (or into an agreement); the user
opens the case and exports it themselves, entering their password so the names go
in at that moment, in their browser.

---

## 1 · Three refusals

**Never write the document yourself.** Not as a file, not as a code block, not as
"here is the draft, paste it into Relex." A draft you produce in chat has no case
to belong to, no version history, no redline, no signature route, no export path —
and it was written without the case's redacted corpus, so it is worse work as well
as unusable work. Drafting runs through the case agent in a steering session
(`relex-steering`). If the user asks for the document here, say plainly that
drafting happens in the case and open a steering session instead.

**Never produce the deliverable locally.** No `.docx`, `.pdf`, `.md`, no writing
to disk, no artifact for the user to download from you. Export exists in Relex
because that is the only place the real names can be put in (§4). If asked, refuse
and hand the export link.

**Refuse personal data the user pastes.** If a name, national ID, address,
date of birth, contact detail, or raw document text arrives in chat: stop, say you
cannot hold it and that it belongs in the case where it is sealed, and hand the
case link. Do not repeat it back, do not "use it just this once," do not summarise
around it. This is not politeness — you are outside the encryption boundary, so
anything pasted here is data the user's own platform was built to keep from you.
The same applies to a user offering to "just tell you the names" so you can draft
faster: the answer is the labels, not the names.

## 2 · What to write instead: the placeholder tokens

The platform substitutes real identities into these tokens at export. Write them
**literally**, in square brackets, with the party's number — this is the whole
vocabulary, and it is defined in `shared/pii/pii_spec.json`
(`canonical_vocabulary.bases`):

| Token | Fills with |
|---|---|
| `[PARTY_NAME_n]` | the party's name |
| `[PARTY_ADDRESS_n]` | their address |
| `[NATIONAL_ID_PLACEHOLDER_n]` | personal identity number (ID card, passport, CNP…) |
| `[TAX_ID_PLACEHOLDER_n]` | tax / VAT registration (CUI, VAT no…) |
| `[COMPANY_REG_PLACEHOLDER_n]` | company registration number |
| `[BANK_ACCOUNT_PLACEHOLDER_n]` | IBAN / account |
| `[PHONE_PLACEHOLDER_n]` | phone |
| `[EMAIL_PLACEHOLDER_n]` | email |

Three things to get right, because each has a failure mode:

- **Always numbered, always this long form.** `[NATIONAL_ID_PLACEHOLDER_2]`, not
  `[CNP_2]`, `[NATIONAL_ID_2]`, or `[NATIONAL_ID_PLACEHOLDER_claimant]`. The
  substitution is per-party-slot; there is **no role-based form and no bare
  un-numbered form** on this path — those resolve to nothing.
- **The token names are jurisdiction-neutral on purpose.** A Romanian CNP, a UK
  NI number and a passport number are all `[NATIONAL_ID_PLACEHOLDER_n]`; a CUI and
  a VAT number are both `[TAX_ID_PLACEHOLDER_n]`. Do **not** invent a token named
  after a local code — it will never resolve.
- **This list is closed.** If a document needs an identifier with no token here,
  leave a typed placeholder for the human (below) and raise it as an open issue.
  A token you invent ships into the client's document as literal text.

You may also *see* a shorter form in documents the user imported — `[NATIONAL_ID_1]`,
`[TAX_ID_1]`, `[BANK_ACCOUNT_1]`, `[PHONE_1]`, `[EMAIL_1]` — because the redactor
that anonymised the upload emits those. The export layer resolves both, so leave
them alone when editing such a document; just write the long form in anything you
compose.

**Dates and amounts are not PII tokens.** They are deliberately not redacted, so
`[DATE_PLACEHOLDER]` / `[AMOUNT_PLACEHOLDER]` are typed placeholders for the human
(below), not party data awaiting substitution.

### These are not the same as typed placeholders

`relex-counsel` also has you write typed placeholders — `[name of client]`,
`[amount in EUR]`, `[date DD.MM.YYYY]` — and the two must not be mixed up:

- **A party token** (`[PARTY_NAME_1]`) means *this person is on the case, sealed,
  and the platform will substitute them at export.* It resolves automatically.
- **A typed placeholder** (`[amount in EUR]`) means *nobody knows this yet.* It
  resolves never; it is an instruction to the human to fill it in.

Writing `[name of client]` where a party token belongs leaves the human retyping a
name the platform already holds — and doing it by hand, outside the seal. Writing
`[PARTY_NAME_4]` for a party that is not attached leaves a literal token in the
document (§3). If the party exists on the case, use its token; if the fact is
genuinely unknown, type a placeholder and say so in your Votum.

## 3 · Get the number right — a wrong one ships into the client's document

Read the roster: `execute GET /ontology/case/{caseId}/participants`. Each sealed
party comes back as

```json
{ "label": "[PARTY_NAME_3]", "role": "claimant", "roleLabel": "...", "description": "...", "ref": "..." }
```

**Copy `label` verbatim into the draft.** The number inside it is the canonical
index the export path resolves against; there is no separate `labelIndex` field to
read, and you must not renumber, reorder, or infer numbers from the order parties
appear in your conversation.

Why this matters more than it looks: **a token whose number matches no attached
party is left literal** — it is never resolved to a positional neighbour, so a
stale reference stays visible instead of silently becoming a different person. The
two outcomes of guessing are therefore a raw `[PARTY_NAME_7]` sitting in a document
that goes to a client, or — if you renumber onto an index that does exist — the
wrong party named in a legal instrument. Read the roster.

The number is the party's **slot**, and it is stable: it is not a positional
counter over however the parties happen to be listed in your conversation, and it
does not shift when a party is detached. The same slot number carries across every
token for that party — `[PARTY_NAME_2]`, `[PARTY_ADDRESS_2]` and
`[NATIONAL_ID_PLACEHOLDER_2]` are all the same person.

## 4 · Export: in Relex, behind the user's password

You never hold the file and you never hold the names. The flow, entirely in the
user's browser:

1. The user opens the document in the case:
   `https://relex.legal/dashboard/cases/{caseId}`
2. They choose the export — **.docx download**, or push to connected storage
   (Google Drive). Signature delivery follows the same model.
3. Relex prompts for their **PII password** and decrypts the parties **in the
   browser**; the tokens from §2 are replaced with the real names and codes at
   that moment.
4. The substituted document is rendered and streamed back **transiently** — the
   server never persists the re-identified version, and the plaintext never
   reaches you.

So the correct answer to "can you send me the signed contract as a PDF" is the
link and the four steps, not a file. Tell them the password prompt is expected and
is what puts the names in — users read it as an error otherwise.

If a format they need is not offered, say so and hand the .docx; do not
reconstruct the document here to fill the gap.

## 5 · Where a document may be produced

- **In a case** — the normal route. Steer the case agent to draft; the draft
  lands in the case with versions, redline, and the export path above.
- **In an agreement** — engagement letters and the intake/e-sign route
  (`relex-intake`); same label discipline, same export.
- **Attached to a case** — analyses, tables, outlines and the named work-products
  in `deliverables.md` are recorded to the case (`POST /cases/{id}/attachments`),
  label-only, and inherit the same export path.

Nothing is produced anywhere else. If none of these fits what the user is asking
for, that is a conversation about scope — not a reason to write it in chat.
