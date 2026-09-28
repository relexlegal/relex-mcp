# Deliverables catalogue (load on demand)

Reached from `relex-counsel` when a matter needs a **named work-product** beyond a
routine draft. Each entry is: *what it is · method · the Relex flow that anchors it
· how to record it to the case*. Everything here obeys the standing discipline —
labels only, citations grounded (`relex-citations`), stop-criteria and the human's
final word (`relex-counsel`). You prepare the deliverable; the human commits any
number, send, or filing.

## 1 · Clause fallback ladder
Per-clause negotiation positions — the firm's preferred wording plus its ranked
fallbacks and walk-away. **Method**: firm playbooks already model each clause as
`{position, fallback}`; read them and apply per clause. **Flow**: `GET /playbooks`
for the firm's positions, apply them during redline on `/cases/{id}/draft`,
recording each deviation from playbook as you go. **Record**: attach the redline +
a deviation log (`POST /cases/{id}/attachments`), label-only.

## 2 · Obligation-extraction report
A table of who-owes-what-by-when across the contract/matter. **Method**: read the
obligation and deadline entities, each fact's proven/alleged status, and their
evidence links from the ontology — the backend now populates these
deterministically. **Flow**: `GET /ontology/case/{id}`; render a table with the
legal deadline **and** the work-back date (`relex-matter`). If the graph lacks
obligations/deadlines, enumerate them from the case context and file the table. **Record**: attach the table; open ontology issues for any gap.

## 3 · Risk heat map
Issues × status × exposure, at a glance. **Method**: cross the case's issues with
their status (`open` / `contested` / `settled`) and a coarse exposure band, from
the ontology — **labels and counts only**, never client figures in chat. **Flow**:
`GET /ontology/case/{id}` for issues + statuses; band exposure from grounded facts.
**Record**: attach the map; reflect any newly-surfaced risk as an ontology issue.

## 4 · Negotiation plan / BATNA
Interests → options → BATNA / reservation point → concession ladder. **Method**:
derive options and the concession ladder from the playbook fallbacks (entry 1);
the BATNA frames the walk-away. **Flow**: `GET /playbooks` for fallbacks; lean on
`the agent-for-legal` negotiation playbooks if installed (`references/interop.md`).
The **human sets the numbers** (reservation, targets) — you structure the plan.
**Record**: attach the plan, numbers left as typed placeholders for the human.

## 5 · Settlement valuation
Expected-value estimate over the grounded issues. **Method**: build an EV tree over
the issues (probability × outcome per branch); **every probability is labelled an
assumption**, never a fact. **Flow**: issues + grounds from `GET /ontology/case/{id}`;
citations for each outcome pass `relex-citations` tiers. Stop-criteria apply — the
human decides whether to rely on it. **Record**: attach the tree with its
assumptions explicit; a Votum noting it is a decision aid, not advice.

## 6 · Examination outline
Cross / direct examination structure. **Method**: defer the examination method to
the `the agent-for-legal` litigation pack (`references/interop.md`) where installed —
it owns the technique. **Flow**: build the outline over the case timeline and
issues; **every quoted document or authority passes `relex-citations` tiers** (no
quote from memory). **Record**: attach the outline; flag any `[verify]` before use.

## 7 · Board / client briefing
A plain-language status for a non-lawyer audience. **Method**: the client-facing
register rules from `relex-matter` — plain language, **no internal vota, no
strategy**, counts and labels only. **Flow**: snapshot + open issues by status +
next deadline + next action. **Record**: attach the briefing; keep the internal
Votum separate from the client-facing text.

## 8 · Multi-jurisdiction comparison
The same question answered per forum, side by side. **Method**: load **one pack per
forum column** (`../jurisdictions/`) and answer each column in that system's own
schema and method — **never blend citation schemas** across columns. **Flow**: per
column, discover + ground via `relex-research`; cite in that forum's form. **Record**:
attach the comparison table; note the forum-selection question as an open issue if
the governing forum is not yet locked.
