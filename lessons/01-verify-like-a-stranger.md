# 01 — Verify like a stranger

**The single most repeated failure across all six products:** a feature ships as code, works for the
builder, and breaks on the first real user. It happened on every project, independently.

## The incidents

- **Localyze, launch day:** the free-tier "Fast" model took **52.6 seconds to answer "Say hello"**
  (0.0 tok/s) on the real device class users actually own. Every internal test had run on better
  hardware or the paid model. Free users' first experience was a broken product.
- **FilingLens:** anonymous users asking the AI got an instant **401** instead of the three
  free questions the design promised. The endpoint was only ever tested logged-in.
- **FilingLens, the expensive one:** the signup flow demanded a card at checkout. Result:
  **0 trials and 0 paid conversions across 781 landing-page views** — not low conversion, *zero*.
  Nobody had walked the funnel as a skeptical stranger with no card in hand.
- **Filing Monitor:** the first real user test hit a report that took 5+ minutes while the spinner
  promised "~1 minute" with no progress feedback. That user's verdict was the whole beta.
- **Ledger Report:** all 50 non-final scenes had identical outcomes regardless of choice — a
  disguised linear rail. Compiled fine, played fine for 10 minutes, hollow for anyone who replayed.

## The rules

1. **Walk the stranger path before calling anything done.** Logged out, no cookies, no card, cheap
   device, first click. If the product has a funnel, walk every step of it and record what you saw.
2. **A green compile is not a working build.** Neither is a green unit-test run. The bar is: the
   exact artifact, on the real path, produced the real outcome.
3. **Test the *worst* supported hardware, not the best.** The dev machine is always top-decile.
   If the product claims to support a device class, run it there before shipping.
4. **Long operations need honest progress.** If work can take 5 minutes, never promise 1. Decouple
   and poll; show real state. A silent spinner is a bug even when the work eventually finishes.
5. **Numbers on screen get eyeballed, not asserted.** A ROE of 110% rendering as "1.1%" passes every
   type check. Screenshot the page and *read* it. For mobile: assert `innerWidth == 390`, full-page
   screenshot every data section, and treat viewport ≠ device-width as a failure, not a footnote.
6. **When a user reports "it's broken" and your tests are green, the user is right.** Your test is
   measuring the wrong thing. Reproduce their exact context before touching code.

## The cheap habit that catches most of it

Before declaring victory, spend five minutes as a hostile first-time user in a fresh incognito
profile on the deployed artifact. Every zero-conversion disaster above would have been caught by
exactly this.
