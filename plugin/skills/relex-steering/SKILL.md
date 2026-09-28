---
name: relex-steering
description: Use whenever substantive Relex case work must be produced — drafting, research, re-reasoning, or any multi-step task on an active case. You produce the legal reasoning yourself. Relex files the document, fetches official text, and keeps the anonymized case. Also covers eval-mode restraint and the support-not-admin rule.
---

# You decide; Relex files

**The rule (canonical — other skills point here):** you solve the user's problem. You lead. Relex is the workspace: read `GET /cases/{caseId}/context` and the ontology, cache official text with `POST /research/scrape`, and file the document you wrote with `POST /cases/{caseId}/draft`.

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

Write each turn as the work you are filing, not as a request for a decision:

- **Goal** — the document or fact this turn files.
- **Constraints** — jurisdiction pack locks, deadlines, addressee/register, locked citations.
- **Already read** — context, ontology, and cached text you are using.
- **Text** — the document you wrote. File it with `POST /cases/{caseId}/draft`.

Bad: "look into the limitation issue." Good: a dated limitation analysis you
wrote from the cached §438 BGB text, with the triggering event, the period,
and the expiry date, filed into the case.

## Reading the steering block

Each `case_req` reply carries `response.steering` — what the workspace has on file:

- `platform_guidance` — how this workspace call works.
- `missing_data` — what is not on file yet. `how:"mcp"` → you fetch it;
  `how:"browser"` → the user adds it on the case page. Map each item onto the
  `relex-ontology` gap taxonomy.
- `suggested_next` — the next read or file call.
- `agent_state` — mode, phase, and what is blocking a draft.

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
they bind every steering turn. You decide, Relex files, the humans
approve.
