---
name: relex-research
description: Use when a Relex case needs legal authority — statutes, case law, regulator guidance, local ordinances — in ANY jurisdiction. Teaches the division of labour — you DISCOVER authorities with your own search and public legal MCPs; the Relex harness FETCHES-AND-CACHES verbatim text so drafts can cite it — plus per-jurisdiction channels and compliance limits.
---

# Legal Research: You Discover, the Harness Grounds

A Relex draft may only cite law whose **verbatim text sits in the global cache**
— the server's verifier rejects anything else. Your web results and memory can
never ground a citation; they only *find* what to ground. So:

```
discover (you: web + public legal MCPs)
  → directive (POST /research/scrape — the harness caches VERBATIM text)
    → ground (you cite only the cached verbatim text, and you decide what it means)
```

Epistemic discipline while discovering is `relex-citations` — in short: no
docket numbers, margin numbers or quotes from model memory, ever.

## Step 1 — know what the harness can fetch

`execute GET /research/sources` — the live registry. `enabled: true` sources
can be fetched-and-cached by directive. Everything else is **discovery-only**:
you research it yourself and pass the best official URL as `sourceHint`, or the
harness falls back to its generic fetch ladder.

## Step 2 — discover (your job)

Use your own WebSearch/WebFetch and, when connected, public legal MCPs. Prefer
official/free sources; identify the authority in **anchor form** — topic, court/
bench, date if known, official source to verify in — never a memorized citation.

| Jurisdiction | Best discovery channels |
|---|---|
| US | CourtListener official MCP (mcp.courtlistener.com) — case law, dockets, citation check; GovInfo/eCFR for federal statutes/regs; LOCUS for local ordinances (below) |
| DE | NeuRIS (rechtsinformationen.bund.de) — new official statutes+case-law API; gesetze-im-internet |
| CH | entscheidsuche.ch (+ its MCP) — BGer + all cantons; Fedlex |
| UK | legislation.gov.uk (open API); Find Case Law (National Archives) |
| FR | justicelibre MCP (no key, ~3M decisions); Légifrance |
| IT | normattiva.it / dati.normattiva.it; SentenzeWeb (last ~5y Cassazione, free) |
| ES | BOE consolidated-law API (statutes) |
| RO | legislatie.just.ro; rejust.ro (decisions); portal.just.ro (dockets) |
| EU | EUR-Lex + CELLAR SPARQL (no auth); CURIA via ECLI |
| CA | CanLII (metadata/citator via API), laws-lois.justice.gc.ca |
| JP | e-Gov Laws API; courts.go.jp (selected decisions) |
| AU | AustLII (read, don't bulk-scrape); legislation.gov.au |

Per-jurisdiction method, citation schema and source whitelists live in the
`../jurisdictions/<XX>.md` packs when installed.

**Compliance limits you must respect during discovery** (encode in your plan):
UK Find Case Law bulk/computational use needs a licence — read individual
decisions, let the harness handle caching lawfully; Spain CENDOJ prohibits bulk
reuse (read single decisions only); CanLII full text is site-only per its terms;
AustLII discourages scraping. When a channel is restricted, discovery stays
manual-and-targeted and the `sourceHint` you pass is the official page.

## Step 3 — directive (the harness's job)

`POST /research/scrape` grounds one authority — params via `search`
(`jurisdiction`, `code`, `article`, `authorityType` = statute | case_law |
regulator | official_act, `sourceHint?`, `caseId?` for provenance; results are a shared cache).

- **Cache-first**: `status: "cached"` means it's already grounded — done.
- `status: "enqueued"` → poll `GET /research/scrape/{jobId}` until `ingested`,
  then cite that cached text in the document you file.
- **An act that is not a code article** (ordinance, gazette decision): `authorityType: "official_act"` and `sourceHint` the official URL or the gazette date. `not_found` means that URL did not yield text; try the next official URL.
- `status: "cooldown"` → a recent fetch failed; give a better `sourceHint` later.
- **Targeted only**: directives are for the citations a draft actually needs
  (the case's `pending_citations` / your audit's grounding gaps). Never bulk.
  There's a modest daily cap; case law behind rate-limited APIs (CourtListener
  free tier) makes every directive count.
- **US case law is now harness-groundable**: `us_courtlistener` is a provisioned
  source, so `POST /research/scrape` with `authorityType: "case_law"` fetches the
  opinion's verbatim text directly (no HTML-ladder fallback). Always re-check
  `GET /research/sources` for the live enabled set rather than assuming.
- **Shared quota, not per-user**: the CourtListener token is ONE credential for
  the whole deployment — every directive across every user draws from the same
  free-tier bucket (as of the May 2026 policy change: 5/min, 50/hr, 125/day).
  This is a stronger reason to keep directives targeted (above): a burst of
  bulk requests from one case can throttle every other case's case-law
  grounding for the rest of the day.
- For case law, put court in `code` and docket/ECLI/reporter cite in `article`
  (e.g. `code: "Cass. 1re civ.", article: "21-12.345"`), `authorityType:
  "case_law"`, and the decision URL you found as `sourceHint`.

## LOCUS — US local ordinances (two vantage points, one corpus)

- **Grounding path (backend)**: `execute GET /research/locus?state=CA&city=…&query=…`
  → discovery results with `coverage`: `matched` / `covered_no_match` (broaden
  terms) / `not_in_locus` (fall back to web; NEVER invent an ordinance). Then
  `GET /research/locus?locusId=…` fetches the section's verbatim public-domain
  text for citation-grade grounding.
- **Analysis path (yours, optional)**: the LOCUS dataset
  (`huggingface.co/datasets/LocalLaws/LOCUS-v1`, paper "Freeing the Law with
  LOCUS") also carries analytical layers (topic, function, enforcement-
  discretion, opacity, paternalism scores) under **CC-BY-NC-4.0**. You may read
  those yourself (HF datasets-server `/rows`) to shape strategy — keep that
  analysis on your side (it is non-commercial-licensed; the Relex backend never
  serves it), attribute the corpus, and still ground any citation through the
  backend path above.
- Read coverage from `GET /research/locus` yourself before you search again.
  `not_in_locus` means that ordinance is not in the corpus. Find the official page and pass it as `sourceHint`.

## Anti-patterns

- Citing from your web page reads ("the court held… [link]") in a draft — the
  verifier will reject it; issue the directive and cite the cached text.
- Directive-spamming a jurisdiction "to have it all" — targeted grounding only.
- A restricted source stays discovery-only. Pass `sourceHint` for an official page.
