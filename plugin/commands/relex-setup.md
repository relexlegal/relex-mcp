---
description: Set up your Relex practice workflow — sign in, set your PII password, add your know-how, let Relex auto-create your encrypted parties, and (for firm owners) set up the organization and client intake.
---

# /relex-setup

The user said something like *"set up my practice workflow with Relex"*. Run the
guided, deep-link-first onboarding. Drive it from the live status endpoint — never
guess where the user is. Follow the PII discipline in the `relex` skill at every
step (PII, documents, payments, and exports happen in the browser, never in chat).

## How to drive it

1. **Make sure you're connected.** Issue any tool call (e.g. the status read in
   step 2). If you are not signed in, the Relex MCP server replies with an OAuth
   challenge and your client opens the user's browser to sign in with Google or
   Apple and approve access — **no key paste**. Tell the user: "A browser window
   will open — sign in to Relex and approve, then come back." Wait for them.

2. **Read progress** with `execute` → `GET /onboarding/status`. It returns only
   anonymized counts, flags, deep links, and the connected account (never PII):

   ```
   { connected,
     account: { uid, plan, planName },   // opaque uid + tier only — never an email
     piiConfigured,
     knowledge: { total, indexed, processing, awaitingParties, failed },
     detectedParties,
     parties: { count },
     organization: { exists, id, role, isAdmin, isOwner, vaultConfigured },  // opaque id + role, no name
     partner: { isPartner, canAcceptPayments, onboardingComplete, published },
     canStartCaseNow,                            // always true — a case is never gated
     nextStep,
     steps: [ { key, done, applicable, actionable, deepLink } ],
     deepLinks: { pii, knowledge, organization, orgVault, partner, partnerStatus,
                  parties, cases, agreements } }
   ```

   **Private data never crosses to you here.** Over your connection (OAuth /
   API key) Relex returns only anonymized data and opaque IDs — never an email,
   a person's name, an organization name, or an address. `account.uid` is an
   opaque id; you cannot see the user's email. Real account details live only in
   the browser.

   **Every flag is instant and authoritative.** `piiConfigured` flips to `true`
   the moment the user saves their password — there is **no propagation delay** and
   nothing to "sync." Never tell the user to wait for a flag to propagate or save.
   Use `nextStep` to decide what to do, then re-read after the user acts.

3. **Act on `nextStep`** (one step at a time; wait for the user, then re-read status):

   - `set_pii_password` → "First, let's protect the personal data in your matters.
     Open **{deepLinks.pii}**, set a password, and save your recovery key. It
     encrypts every name, ID, and document in your browser — I only ever see
     labels like `[Party 1]`." Wait, then re-read.

     **If it's still `false` after they say they've set it:** it did not save on
     *this* account. The overwhelmingly common cause is the browser being signed
     into a **different account** than the one this the agent is connected to. You
     cannot see their email (it never crosses to you), so tell them plainly:
     "Make sure you set the password while signed into relex.legal as the **same
     account you used to connect the agent**." Then re-check **once**. Do not invent
     a delay; the flag is instant.

   - `add_knowledge` → "Now add your know-how. Open **{deepLinks.knowledge}** and
     upload your playbooks, templates, and past matters. Relex indexes them
     privately, builds your personal (and, in a firm, your organization) knowledge
     model, and reads out the parties it finds." Wait, then re-read.

   - `processing` → indexing/OCR is still running. Tell the user it's processing,
     wait a bit, and re-read. Don't loop aggressively — check back when they say
     they've uploaded, or after a short pause.

   - `finish_parties` → Relex found parties in the knowledge but they aren't
     created yet (PII is probably locked). "Open **{deepLinks.knowledge}** with
     your PII password unlocked — Relex will auto-create the {detectedParties}
     parties it found, encrypted in your browser." Wait, then re-read; confirm
     `parties.count` went up.

   - `setup_organization` → the user is a firm **owner/administrator**
     (`organization.isAdmin`) and their firm has no shared PII vault yet. "Set up
     your organization vault so your whole firm's client identities are protected
     under one key and every member can work on them. Open **{deepLinks.orgVault}**,
     pick the organization, and create the org vault." Manage members/roles at
     **{deepLinks.organization}**. Wait, then re-read; confirm
     `organization.vaultConfigured`.

   - `register_partner` → to **intake paying clients** the user registers in the
     partner program (set up the org vault first, so client PII seals to the org
     key). "To take on clients and accept payments through Relex, register in the
     partner program: **{deepLinks.partner}**. Once verified you can send priced
     offers and get paid; without it you can still send agreements and invoices,
     but you'd collect payment yourself." Check status at **{deepLinks.partnerStatus}**.
     Read `partner.canAcceptPayments` to know if payouts are live.

   - `create_case` → setup is done. Confirm using the anonymized counts, then offer
     to start the first case (see the `relex` skill: `POST /cases` with an **empty
     body**, then the eval flow names and tiers it). On `payRequired`/`402`, hand
     **`…/dashboard/cases/{caseId}`** to review the offer and pay — never quote prices.

4. **A case is never gated.** `canStartCaseNow` is always `true`: PII password,
   knowledge, org, and partner all *protect and enrich* the work but none is
   required to open a case. If the user just wants to start, start — offer the
   setup steps alongside, not as a blocker.

5. **After the first case — agreements lifecycle.** When the user invites a partner
   into a case, or an organization invites a guest client, the case carries
   agreements: draft → send → e-sign → track to completion/renewal. Point to
   **{deepLinks.agreements}** and see the `relex` skill for the case/agreement flow.

6. **Report progress in the chat** as you go (e.g. "✅ PII password set",
   "✅ 4 parties created from your knowledge") using the anonymized counts — never
   echo a name or ID. If a step stalls, hand the relevant deep link again and wait.

Keep it warm and short. The whole point: the user logs in once, and you walk them
through a private setup where their know-how powers the work and their clients'
identities never leave their browser.
