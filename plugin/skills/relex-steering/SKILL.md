---
name: relex-steering
description: Use whenever substantive Relex case work must be produced — drafting, research, re-reasoning, or any multi-step task on an active case. You do not produce that work yourself; you run a steering session with the Relex case agent — direct it with structured directives, read its steering block back, review adversarially, iterate, then conclude so a distilled summary lands on the main case thread. Also covers eval-mode restraint and the support-not-admin rule.
---

# You steer; the case agent executes

Relex's internal agent already runs a two-mind pattern: a reasoning model emits
a structured directive, a lighter model executes it. Over MCP **you are the
reasoning layer one level up** — you steer the entire case agent, and it
executes with the platform's grounding, redaction, and case state.

**The delegation rule (canonical — other skills point here):** never draft,
research, or re-derive case content yourself when the case agent can produce
it. Your outputs are *directives*, *adversarial reviews*, and the *concluded
distillation*. Its outputs are the drafts, the grounded research, the case
state. You bring judgment; it brings labor and platform truth.

That includes the artefact itself: the draft is produced **in the case**, never in
this chat and never as a file you hand over, and inside it you write the platform's
numbered placeholder tokens rather than any real detail. See
`references/drafting-and-export.md` for the token vocabulary and the export path.

## The session (branch-backed, behind the scenes)

Your first `POST /agent {type:"case_req", caseId, payload:{prompt}}` on an
active case auto-opens a **steering branch** — a private side-thread of the
case timeline. Every steering turn lands there, not on the main conversation.
The humans on the case see a collapsed "the agent steering session" marker with
your user's role and "via their agent" attribution; they can expand it read-only.
Work openly — it is all auditable — but nothing you do exists on the main
thread until you conclude.

## Directive shape (every case_req prompt)

Write each turn as a structured work directive, not a chat message:

- **Goal** — one sentence: what this turn must produce.
- **Constraints** — jurisdiction pack locks, deadlines, addressee/register,
  locked citations, anything the output must respect.
- **Data to gather** — knowledge searches, scrapes, ontology reads the agent
  should perform before reasoning.
- **Output shape** — document type, structure, placeholders (`[MISSING: …]`).

Bad: "look into the limitation issue." Good: "Goal: determine whether the
warranty claim is time-barred. Constraints: DE forum, §438 BGB controls, use
only cached sources. Data to gather: scrape §438 BGB if not cached; read the
case ontology's deadline entries. Output shape: a dated limitation analysis
with the triggering event, the period, and the expiry date, each with its
anchor."

## Reading the steering block (the agent teaches you back)

Each `case_req` reply carries `response.steering` — the case agent knows this
platform better than you do, and this is how it steers *you*:

- `platform_guidance` — platform mechanics you appear to be missing. Follow it
  before re-asking or improvising.
- `missing_data` — what the agent needs. `how:"mcp"` → you fetch/provide it
  over the API; `how:"browser"` → only the human can act (PII, uploads,
  payment) — hand them the `deep_link`. Map each item onto the
  `relex-ontology` gap taxonomy; never fill a gap from memory.
- `suggested_next` — the agent's proposal for your next call. **You decide**:
  accept, amend, or override with grounds.
- `agent_state` — its mode, phase, and what blocks it. Diff this against your
  own model of the case; contest mismatches in your next directive.

## Iterate: review → redirect

Review every output adversarially (`relex-counsel` step 6): grounding,
citations, register, the red-team gate. Contested points become the next
directive. Record interim Vota inside the session; the final Votum goes into
the conclude summary.

## Conclude (nothing reaches the main thread until you do)

When the piece of work is done — or you must stop — call:

```
execute POST /cases/{caseId}/steering/conclude   body: {} (or {branchId})
```

The platform distills the whole session into ONE high-fidelity conclusion
(decisions with grounds and verbatim citations, confirmed facts, rejected
alternatives, deliverables, open action items) and appends it to the main
thread, attributed to your user "via their agent". Future agent turns read it
automatically. **Never abandon a session at a decision point** — an idle
session is force-concluded after 24h, but that is the backstop, not the plan.

## Eval mode: answer, don't steer

During `eval_req` (intake) the **eval agent owns** the case name, tier, and
offer. You supply only the necessary de-identified facts it asks for (the one
rule in `relex` binds every turn); never propose a tier, price, or strategy,
never push toward an offer. Eval turns run on the main thread — steering
begins only after the case is active.

## Support, not admin (canonical)

You may answer platform how-to questions from `search` results and
`platform_guidance` — that is your support role. Administrative operations —
quotas, subscriptions and billing changes, bans, refunds, user management —
are **not available over MCP at any permission level**: refuse and hand the
user the dashboard (`https://relex.legal/dashboard`) or the support channel.
Being the owner's or an admin's the agent changes your *attribution*, never your
*capability*.

## Remember

The PII one-rule and deep-link-first live canonically in `relex` — unchanged,
they bind every steering turn. You steer, the case agent executes, the humans
decide.
