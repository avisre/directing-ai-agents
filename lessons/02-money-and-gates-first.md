# 02 — Money and gates are features, not afterthoughts

Two mirror-image failures recurred on every monetized project: the **pricing layer shipped as a
stub**, and the **abuse/security gates shipped last** (or not at all until an adversarial review).

## Monetization shipped broken

- **Ledger Report:** ~1,050 installs, top country Brazil, IAP priced in auto-converted USD because
  regional pricing "would be set later." Revenue across the entire install base: **$0. Zero buyers.**
  The audience the algorithm delivered could not afford the price the stub charged.
- **FilingLens:** the card-at-checkout trial wall produced **0 trials on 781 views**. The
  fix (no-card 7-day trial, upgrade route, in-app banner) took one day to build — after weeks of
  traffic had already bounced off the wall.
- **Localyze:** paywall shipped while the paid products didn't exist in the store console, so paying
  was literally impossible; then everything was made free without ever measuring willingness to pay.

**Rules:**
1. The pricing path is walked end-to-end (real checkout, real geography, real currency) before any
   traffic is pointed at the product. A funnel with an untested toll booth converts at exactly 0%.
2. Price for the geography you're actually acquiring, not the one you imagined. Check the install
   country breakdown *before* setting price.
3. If you defer monetization, that's a decision with a date on it — not a stub that quietly becomes
   permanent.
4. **Walk the payout path, not just the pay-in path.** A marketplace launch was declared "complete"
   with the listing live and redemption webhooks validated — while the payout onboarding (payment
   processor contact form, bank details, tax forms) sat unfilled in an unclicked Billing tab. Trace
   one imaginary sale end-to-end: buyer pays → platform → *your bank account*. Every hop needs to
   exist before you call a launch done, and the last hops are the easiest to forget because no test
   fails when they're missing.

## Gates shipped last

- An adversarial review of the agent-cost tower found **11 security/correctness issues** in "done"
  work — including a notification channel that trusted a public broadcast topic (anyone who guessed
  the topic name could approve budget overrides).
- FilingLens had tier-leak UX bugs (gated features visible to the wrong tier), an unset
  admin token on prod, and an AI endpoint with no quota wall — each found *after* the feature was
  declared complete.
- Signup forms attracted bot abuse in production; honeypot + blocklist + rate limit were retrofits.

**Rules:**
1. Every feature that costs money per use (LLM calls, emails, API quota) ships **with** its limiter.
   No exceptions — an ungated AI endpoint is a donation to whoever finds it first.
2. Every gate needs a deny-path test: prove the 401/429/upgrade-wall actually fires, and that it
   survives a process restart.
3. Schedule an adversarial review before prod exposure: one pass whose only job is to break in,
   leak tiers, and spend your money. It will find things the builder cannot see, because the builder
   is optimizing for the feature working.
4. Secrets live outside the repo, always — especially if repo visibility can ever change. Assume
   every repo goes public eventually.
