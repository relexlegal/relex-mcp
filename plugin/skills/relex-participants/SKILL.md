---
name: relex-participants
description: WHO is on a Relex case — sealed legal parties vs app participants (lawyers, staff, guests, partners), always as labels, never names. Essential when working a case from a shared Slack channel with the agent tagged in. Teaches who's-who, the two never-joined name-spaces, and channel-to-case binding.
---

# Who's Who on a Case (labels only)

A legal matter has two kinds of people, and Relex keeps them in **two separate
name-spaces that you never join**:

- **Sealed legal parties** — the client, the opposing side, witnesses, third
  parties. Their real identities are encrypted; you only ever see the tokens
  `[PARTY_NAME_1]`, `[PARTY_NAME_2]`, … each with a role and a short description.
- **App participants** — the people *using* the workspace on this case: the
  owner, practice members, invited guests (often the client), and outside
  partners. You see `[OWNER]`, `[MEMBER_1]`, `[PARTNER_1]`, `[GUEST_1]`, … each
  with a kind, an access level, and a description — never a name, email, or photo.

The design borrows the *shape* of a zero-knowledge circuit — each side proves
what it needs without handing over the raw identities — but it is not literal
zero-knowledge crypto: it is label-first minimal disclosure. You reason over the
labels; the real names live encrypted and are revealed only in the browser.

## 1 · Read the roster first

Before you draft, advise, or answer "who is …", read the roster:

```
execute({ method: "GET", path: "/ontology/case/{caseId}/participants" })
```

Returns labels only: `{ parties[], participants[], counts, deepLinks }`, e.g.

```
parties:       [ {label:"[PARTY_NAME_1]", role:"claimant",  description:{…}},
                 {label:"[PARTY_NAME_2]", role:"defendant", description:{…}} ]
participants:  [ {label:"[OWNER]",     kind:"owner",   access:"owner"},
                 {label:"[MEMBER_1]",  kind:"member",  access:"write"},
                 {label:"[GUEST_1]",   kind:"guest",   access:"read"} ]
```

Render it as `label · role/kind — description`. For the full web of relationships
(who represents whom, which party a claim is against), pull the graph via
`relex-ontology` (`GET /ontology/case/{caseId}`) — the `[PARTY_NAME_n]` tokens
are the same in both, and are stable for the life of the case.

## 2 · The two name-spaces — never joined

This is the one rule. In a shared Slack channel the agent Tag legitimately knows the
**workspace** ontology: teammates' real names, channels, threads. Relex holds the
**case** ontology: who's who legally, sealed. Keep them apart.

- A teammate's real name in the channel (`@dan`, "Ana from our side") is the
  workspace ontology — fine to use for *them*.
- A sealed party (`[PARTY_NAME_2]`) is the case ontology — you answer about it in
  labels, and you **never assert that a label IS a named person**, even if a
  human in the channel says so. "Is [PARTY_NAME_2] our client Mrs. Ionescu?" →
  you don't confirm or deny; you keep working in labels and offer the reveal link.

Composing the two — "the claimant [PARTY_NAME_1], whom [MEMBER_1] represents" — is
exactly the useful join. Joining *identities* — "[PARTY_NAME_1] = <real name>" —
is the one you never make.

## 3 · When someone types a real client name

People will. Handle it cleanly:

- **Don't repeat it, store it, or write it back** to Relex — the server refuses
  any `execute` that would move party PII and returns a deep link (the one PII
  rule, canonical in `relex`; the refusal is the correct path, not an error).
- Answer in labels: "I've got the client as `[PARTY_NAME_1]` (claimant) — I'll
  keep it labelled here; open the Parties tab to see or edit the real identity."
- Offer `deepLinks.parties` — the person reveals identities in their own browser,
  behind their own PII password. It never comes back to the channel.

## 4 · Binding a channel to a case (the agent Tag)

In Slack you must know *which* case the channel is about before you touch data:

1. Look for the case deep link (`…/dashboard/cases/{caseId}`) in the channel
   topic, a pinned message, or the thread you're tagged in.
2. Confirm once — "Working on case «Acme dispute» — right?" — then keep the
   binding in the channel's memory. **Never guess a caseId.**
3. A channel that spans several matters: bind **per-thread**, and re-confirm when
   the thread changes matter. When in doubt, ask for the case link.

A first `search`/`execute` triggers the OAuth sign-in (the admin who connected
the connector); see the main `relex` skill. If a call comes back `401/403`, the
tagger lacks access to that case — say so; don't try another case.

## 5 · Deep links are safe to post

A case/parties deep link carries **no PII** — it's just a caseId. Posting it in a
busy channel is fine: every reader who clicks authenticates as themselves, sees
only what their own access allows, and any real identity is decrypted locally
behind their password. So the roster stays label-only in the channel, and reveal
is always per-person, in the browser.

## 6 · Ambient & deadlines

For standing "who owes what by when" work, stay in labels and lean on
`relex-matter` (deadlines, timeline, comms log). Post reminders as
"`[PARTY_NAME_2]` must respond by 14 May" — never with a name. **Never** do an
ambient reveal, and never make a structural change (attach/detach a party, edit
the case) without the user asking.

## Anti-patterns

- Confirming or denying that a `[PARTY_NAME_n]` label is a specific real person.
- Echoing a client name back into chat or into an `execute` body.
- Guessing a caseId instead of binding to the case link in the channel.
- Joining the two name-spaces into an identity map ("Party 1 = …"). Compose by
  role and relationship; never by identity.
- Treating a server PII refusal as a failure to retry — it's the reveal path.

## Alongside

- `relex` — connect, the connector tools and workflow, the one PII rule.
- `relex-ontology` — the full case graph and the collaboration loop.
- `relex-matter` — deadlines, timeline, conflicts, comms, closing.
