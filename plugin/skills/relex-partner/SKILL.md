---
name: relex-partner
description: Guide a lawyer or firm through joining the Relex partner program — the registration that lets them charge their own clients (fees, invoices) via their own Stripe account. Covers the states (pending, verified, published), payment onboarding, Relex verification, and when client invoicing unlocks.
---

# Joining the Relex partner program

Registering as a partner is what lets a firm **charge its own clients** — issue an
engagement fee or invoice that the client pays by card, straight to the firm.
Relex never holds that money: registration provisions the firm's **own** payment
account, and client charges go directly to it. So client invoicing and paid guest
intake stay locked until the firm is a verified, published partner.

You **guide** the process; the user does every step that touches their details or
payment **in their browser**. Point them to the right page and explain what each
step is for. You never enter their credentials, bank details, or rates.

## The path (point the user to each step)

1. **Register** — `https://relex.legal/partners/register` (a short wizard):
   - Profile: name, type (lawyer / notary / specialist), photo, bio, region.
   - Legal details: bar/registration number, regulator, admission region.
   - Proof of standing: upload a licence/registration document.
   - Consent: agree to the partner terms + screening.
   - Rate card: the firm sets its own fees (you don't set or quote these).
2. **Program subscription** — completed inside the wizard: the firm starts the
   partner-program subscription and adds a payment method. This is a prerequisite
   for the next step.
3. **Payment onboarding** — the wizard hands off to set up the firm's payment
   account (identity + bank details, entered on the processor's secure pages).
   Track progress at `https://relex.legal/dashboard/settings/partner/status`.
4. **Manual Relex verification** — once submitted, a Relex reviewer checks the
   firm's standing (they receive the registration for review). The firm's status
   shows **pending verification** meanwhile. Nothing the user can do but wait.
5. **Published / live** — after Relex verifies and publishes the firm, it is
   **active**: it appears in the partner directory and can now issue client
   invoices and run paid guest intake.

## The states — read them back to the user

- **pending** — registered, awaiting Relex verification. Client invoicing is
  locked. Check `…/settings/partner/status`.
- **verified (not yet published)** — Relex confirmed standing; the firm is one
  step from live. Still awaiting the publish.
- **active / published** — live. Client invoicing + paid guest intake unlocked.

If the firm tries to invoice a client before this, the server refuses with a link
to finish registration or check verification status — relay that link; it's the
correct path, not an error.

## What to tell the user

- Why it's needed: to charge clients directly, the firm needs its own registered,
  verified payment identity — that's the partner program.
- The order matters: subscription → payment onboarding → verification → publish.
  Each unlocks the next; skipping isn't possible.
- Verification is done by a person at Relex, so there's a short wait after
  submitting — that's expected.
- Everything sensitive (documents, bank details, rates, card) is entered by the
  user in their browser. You never handle it.

## Billing → the firm's own ERP

Once live, the firm keeps 100% of what clients pay them (flat monthly, 0%
commission), and Relex hands that money movement to whatever ledger they already
run. This connection is **direct between the firm and Relex — never through you**:
the firm's own ERP or browser session pulls their billing export (client invoices,
case-work earnings, Connect payouts) with the firm's own credentials. There is no
MCP tool for it and there never will be — a partner's aggregate billing history is
exactly the kind of private financial data that must not cross to an agent. Point
the user at `/docs/partner-program/erp-export` and let them (or their ERP) fetch
it directly; don't call it, don't summarize amounts you haven't seen, don't ask
the user to paste rows to you.

## What NOT to do

- Never quote or set the firm's fees, or Relex's own prices — the firm sets its
  rate card; you don't advise amounts.
- Never enter the user's credentials, bank details, or payment info — always the
  browser.
- Don't promise a verification timeline or outcome — it's a manual review.
- Don't retry a "registration required" / "pending verification" refusal — relay
  the link.

## Alongside

- `relex-intake` — once live, the end-to-end client intake (agreement → e-sign →
  invoice) and paid guest onboarding.
- `relex` — connect, the two tools, the one PII rule.
