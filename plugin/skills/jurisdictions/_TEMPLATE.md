# [XX] — [Jurisdiction] jurisdiction pack

> the agent-facing orientation for driving a matter in this jurisdiction. It
> **complements** the Relex backend's agent-facing module (which owns the full
> citation tables, forums, tones and case-type method) — read together, never
> duplicated. Use it to decide WHERE to discover authority, WHAT the harness can
> ground, the COMPLIANCE limits, and the METHOD reminders. Pair with
> `relex-research` (discovery→directive→grounding) and `relex-citations` (locks).

## Legal family & sources of law
One line: legal family; the layering (federal/state/supranational); which layer
governs the common matter types; the default when the sub-jurisdiction is unclear
(and the gap to flag).

## Citation — schema + hard locks + official whitelist
- **Schema**: the canonical citation form(s) (the backend module has the full
  tables — name them, don't restate exhaustively).
- **Hard locks** (jurisdiction-specific): forbidden proprietary locators; the
  pinpoint rule; any renamed/recodified-source traps; framing bans (e.g. no
  stare-decisis argument in civil-law systems).
- **Official free sources whitelist**: the databases a citation may name as its
  verifiable source (statutes + case law).

## Discovery channels (what YOU use to find authority)
Table: statutes | case law | registers — each with the public MCP / API / site
the agent uses for DISCOVERY, and whether it needs a key.

## Grounding availability (what the HARNESS can fetch-and-cache verbatim)
Which authorities `POST /research/scrape` can cache today (check
`GET /research/sources` live), and which are **discovery-only** (compliance or
no-adapter) → you research them and pass a `sourceHint`.

## Compliance limits (encode; do not trip)
Bulk-reuse / computational-analysis / scraping restrictions per source
(licences, CAPTCHAs, terms) — the rules that keep discovery lawful.

## Method notes
The 2–4 reminders that change reasoning here (claim-construction order, burden
rules, controlling-vs-persuasive, style/register conventions, doctrinal bans).

## Community skills to consult
Named community packs for this jurisdiction to lean on for method/citation depth
(via `references/interop.md`), and what each is good for.

## Limitation / deadline heuristics (orientation only — never finalize from memory)
The high-frequency limitation & procedural periods and their classic traps — as
ORIENTATION only, with **no** closing restatement. The deadline RULE itself lives
in `relex-matter` (canonical); a pack carries it ONLY via this exact heading.

## What packs never restate (canonical homes elsewhere)
Keep a pack to what is jurisdiction-specific. Do NOT restate:
- **the deadline rule** beyond this heading — canonical in `relex-matter`;
- **the PII one-rule** — canonical in the `relex` skill + the server `read_api` / `execute`
  tool description at runtime;
- **the interop framing** (how Relex layers grounding + PII custody over community
  packs) — canonical in `references/interop.md`; a pack keeps only its own
  community-pack pointers;
- **backend Method depth** — the backend injects a per-jurisdiction `## Method`
  into its reason model, so a pack's Method notes stay 2–4 reminders for the agent's
  OWN drafting/review, not a full method treatise.
