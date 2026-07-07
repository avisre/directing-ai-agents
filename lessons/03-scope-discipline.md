# 03 — Scope discipline: milestone contracts and close-or-commit

Agents make scope creep *cheaper*, which makes it *worse*. When the marginal cost of "while we're
here, also…" drops near zero, the only defense is deciding scope before starting, not during.

## The pattern, observed

- **Guru portfolios** (FilingLens): started as "10 investors." Shipped as 27 investors with
  verified EDGAR CIKs, daily cache pre-warming, 3-tier access gating, AI analysis, a landing-page
  feature card, and a 20,000-URL sitemap. Each increment was individually reasonable; the sum
  delayed everything else by days.
- **AI avatar:** Phase 1 desktop app → STT/TTS pipeline → Windows CI/CD → Rust port of the cost
  tower → native Bevy VRM renderer → a second mascot app port. Five products' worth of scope,
  zero shipped to a user.
- **Ledger Report story rework:** "two branching tracks for one character" became stat-gating on all
  20 endings, a debt-interest system with era-accurate rates, and a five-way convergence finale —
  built, verified in an emulator, and **never integrated into the live build**.

## The counterweight

Ambition isn't the enemy — one project's charter explicitly says *build the fuller, best-in-class
thing*. The enemy is **undecided** scope: ambition that accretes mid-task instead of being chosen
up front. "Build the ambitious version of X" is a scope. "X, plus whatever seems good while I'm
in there" is not.

## Rules

1. **Write the milestone contract before the first edit:** what ships, what's explicitly out, and
   what "done" looks like (a verifiable check, not a feeling). Ship it. *Then* decide expansion as
   a new milestone with its own contract.
2. **Close or commit within 3 days.** Hard problems (GPU backends, licensing research, narrative
   redesigns) that stall get one of two endings: fully integrated, or written down and shelved with
   an explicit decision. "Locally working, ship in limbo" is the worst state — it costs context
   forever and delivers nothing.
3. **Track open loops in one place.** The retrospective found a dozen half-finished efforts nobody
   was tracking: a 22-agent research conclusion never acted on, store-listing copy never pasted in,
   regional pricing "initiated" but never set. Each was individually small; the pile was a project.
4. **Every changed line traces to the request.** Don't "improve" adjacent code, reformat, or
   refactor what isn't broken while passing through. Surgical diffs keep review possible and keep
   blame (git and human) meaningful.
5. **When the user asks for one thing, deliver that thing first.** Offer the expansion after it
   ships, as a choice — not as a fait accompli inside the same diff.
