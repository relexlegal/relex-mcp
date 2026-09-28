---
name: relex-ontology
description: Use when working a Relex case beyond a single question — auditing what the case "understands", finding gaps or contradictions in the case/firm ontology graph, and deciding what to acquire next (RAG, statutes, case law, documents). Teaches the read → audit → repair → direct-acquisition → converge loop over the Relex MCP.
---

# The Ontology Collaboration Loop

The ontology is the case's **understanding**; the graph is only its data — and it
can be **wrong or incomplete**. Your job (as counsel, see `relex-counsel`) is to
audit that understanding, repair it, and direct the Relex harness to acquire what
is missing. The harness's next reasoning turn automatically reads what you fixed.

```
read → audit → repair/enrich → direct acquisition → you file the document → re-read → converge
```

Everything below is PII-safe by construction: people appear only as
`[PARTY_NAME_n]` label tokens; fact entities carry keys, never values. The
server's PII gate rejects raw identifiers you might try to write — don't.

## 1 · Read

- `execute GET /ontology/case/{caseId}` — the case graph: real-world legal
  objects (parties, obligations, clauses, statutes, events, ISSUES) and the
  verbs binding them (`cites`, `concerns`, `supports`, `contradicts`,
  `undercuts`, `party_to`, `raises`, `established_by`).
- Optional when explicitly enabled: `?view=digest&jev=1` can attach a top-level `jev` block with advisory suggestions. Jev is disabled by default. Retrieve fuller evidence whenever the task requires it, irrespective of `graph_sufficient`; never replace source checks, permissions or user approval with a suggestion. An absent block or a 503 from `POST /jev/decide` does not block the normal workflow.
- `execute GET /ontology/firm` (`?scope=org&id=` for an org) — the practice's
  abstract concept graph (doctrines, clause types, argument patterns). No case
  instances, no PII.
- Read the case itself too (`GET /cases?caseId={caseId}&full=true` — caseId is a
  query param; `full=true` is REQUIRED to get the timeline/phases, a plain
  `GET /cases` returns a bounded summary): timeline, phases, locked
  issues, drafts. The graph must MATCH the case data — mismatch is a finding.

## 2 · Audit — formal conflicts first, then the gap taxonomy

**The system audits with you.** Every ontology read carries `conflicts[]` —
formal integrity findings the platform derives fresh from the graph on every
read (never stored, so never stale): unresolved `contradicts`/`undercuts`
edges, claims resting on ALLEGED facts, ungrounded claims, deadlines without
date or trigger, obligations without obligor, open issues with no acquisition
plan, duplicates, low-confidence structure, entities flagged stale, and a
whole-graph staleness flag when case data changed after the last ontology
update. The digest view and your steering block (`steering.conflicts`) carry
the top findings; `?view=full` (or the plain GET) has the complete list with
`resolution` hints. The user sees the SAME list in the case UI.

Work them symbiotically: **resolve** each conflict via §3 ops (settle/contest
the issue, add the missing link/date/party, merge duplicates, re-verify and
clear a stale flag) — and **contribute** conflicts the formal checks cannot
see: when YOU spot a semantic contradiction, add the `contradicts`/`undercuts`
edge; when something presupposes outdated law, set `properties.stale=true` on
it. The moment you write it, the system formalizes your finding and shows it
to the user and the internal agent too. Never leave a conflict silently
unaddressed: resolve it, acquire what resolves it, or escalate both sides to
the human.

### The gap taxonomy

Walk the graph against the case facts. For each gap type there is one correct
acquisition verb — never fill a gap from model memory:

| Gap | Signal | Directive |
|---|---|---|
| **Grounding** | an issue/claim cites no statute, or cites one with no cached verbatim text | `POST /research/scrape` (see `relex-research`) |
| **Coverage** | a fact pattern with no enumerated candidate issue at all | enumerate the issue from the case context and file it on the ontology |
| **Entity** | an actor appears in the timeline/docs but not as a `[PARTY_NAME_n]` node | user finishes pending parties in the browser (deep link) or id-only attach |
| **Evidence** | a fact asserted with no source document | deep-link the user to upload; or search knowledge via their agent |
| **Conflict** | `contradicts`/`undercuts` edges nobody resolved | acquire the resolving authority, or escalate to the human with both sides |
| **Temporal** | timeline holes around dispositive events | ask the user ONE targeted question (question-brake, `relex-counsel`) |
| **Currency** | statute possibly amended / transposition pending | re-scrape the latest version before relying on it |

## 3 · Repair / enrich

`execute POST /ontology/case/{caseId}/agent` with a natural-language
instruction; the editor turns it into typed, transactional, PII-gated ops
(upsert, merge, link, retype, set issue status):

```
execute({ method: "POST", path: "/ontology/case/abc123/agent", body: { message:
  "Add issue 'limitation period' status=contested; link it as undercutting the
   payment claim; merge the duplicate 'Delivery Contract' entities; the
   [PARTY_NAME_2] node is the addressee institution — retype it authority." } })
```

Rules:
- Write only what case data or **verified** sources support — never model memory.
- ISSUES carry status `open | contested | settled`. Mark `settled` only when
  grounded (cached verbatim text or a user-confirmed fact) — say the ground in
  the instruction so it lands in the graph.
- After big case-state changes, `POST /ontology/case/{caseId}/reconcile` rebuilds
  the deterministic backbone (additive; your edits survive), then re-read.
- Curate reusable patterns (clause types, argument shapes — nothing
  case-specific) into the firm graph: `POST /ontology/firm/agent`.

## 4 · Direct acquisition

You discover; the harness fetches-and-caches (division of labour —
`relex-research` has the full doctrine):

- `GET /research/sources` — what the harness can fetch verbatim vs discovery-only.
- `POST /research/scrape {jurisdiction, code, article, authorityType, sourceHint?, caseId?}`
  — cache one authority's verbatim text. Poll `GET /research/scrape/{jobId}`.
- `POST /cases/{caseId}/draft` with `draftName` and `content` — file the document
  you wrote. Use label tokens from the participant roster.
- `POST /agent {type:"case_req", caseId, payload:{prompt}}` — a workspace turn.
  The reply's `steering` block lists what is on file and what is missing
  (`relex-steering`). You supply any text to file.

## 5 · Converge — and say so

The loop is done when:
1. every locked/settled issue is **verbatim-grounded**,
2. every open issue has an **acquisition plan** (directive issued, question
   asked, or escalated),
3. no unresolved `contradicts`/`undercuts` edge remains without a human flag,
4. addressee and deadlines are locked in the case data.

Record the state as a short **Votum** (interim vote) on the case via a
`case_req` message — e.g. *"Ontology audit: 6 issues grounded, 1 contested
(limitation — directive pending job 4f2c), no open conflicts; addressee locked.
Next: re-reason after ingest."* Interim Vota land on the steering branch;
capture the convergence state in the session's **conclude summary**
(`relex-steering`) so it propagates to the main thread. Report progress to the
user with counts and labels only — never names.

## Anti-patterns

- Filling a graph gap from memory ("I know Art. 823 says…") — that's a
  **grounding gap**, not knowledge. Issue the directive.
- Re-doing the harness's work: you never scrape, never RAG-search private
  corpora yourself, never re-implement its reasoning directive.
- Overwriting agent-locked issues without evidence — repair with grounds, or
  contest with an `undercuts` edge and acquire.
- Big-bang rewrites of the graph. Small typed ops, then re-read.
