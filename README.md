# Directing AI Agents: a founder's playbook from six shipped products

Five months. Six shipped products. Nearly all of the hands-on work done by AI under one person's
direction. This repo is the complete, honest write-up of what that actually takes — the system,
the failures, the prompts that worked, and the ones that quietly cost weeks.

**The products** (all real, all shipped):

| Product | What it is | Status |
|---|---|---|
| FilingLens | AI stock research SaaS — every answer sourced from SEC filings | Live; lifetime-deal launch |
| Localyze | Offline AI assistant (Android + Windows/macOS/Linux) | Live on web; Play Store |
| The Ledger Report | Narrative strategy game, 1947 India, 11 languages | ~1K installs on Google Play |
| Discover Hinduism | KMM prayer/lore app, 4 locales, 381 generated illustrations | Live on Google Play |
| Snapdragon NPU recipe | LLMs on "unsupported" Hexagon v69 — 31.3 tok/s measured | Published research |
| Control Tower / avatar | Desktop agent-cost monitor + VRM avatar | Prototype |

**The three documents:**

- **[twitter-article.md](twitter-article.md)** — the story version (fits X's 25k-char long-form limit)
- **[projects.md](projects.md)** — project-by-project: what each product taught, incident by incident
- **This README** — the exhaustive version: the full operating system for directing AI agents

---

## Part 1 — The org chart: two models, two jobs

The highest-leverage decision wasn't a prompt. It was separating **building** from **judging** and
assigning them to different models with different standing orders.

### The builder: Claude Opus 4.8

Writes the features, scripts, migrations, tests. Gets clear specs, bounded scope, and full context
for the task at hand. Optimized for throughput. The overwhelming majority of tokens are spent here.

### The supervisor: Claude Fable 5

Anthropic's Mythos-class model, priced at 2× Opus. It does not write much code. Instead it:

- **Audits** the builder's output before anything ships
- **Executes irreversible actions** — production deploys, payments/KYC flows, anything public-facing
- **Verifies with evidence** — renders documents to pixels, curls production URLs logged-out,
  screenshots viewports — under a standing order to claim only what it has personally seen
- **Maintains the memory system** (Part 3) and writes new rules when corrected

### Why pay double for the model that writes less?

Volume and judgment concentrate in different places. ~90% of tokens are code; ~90% of disasters
are judgment (a wrong deploy, a corrupt tax form, a funnel wall nobody walked). Cheap tokens where
the volume is; expensive tokens where mistakes are irreversible. The supervisor catches something
real approximately weekly. One catch (Part 2, incident 1) was worth 30% of all future revenue.

### The delegation contract

The supervisor delegates coding to builder subagents with a precise spec, then **verifies the
result independently** — it re-renders, re-runs, re-reads. The builder's own success report is
treated as a claim, never as evidence. This is not pessimism about the builder; it is structural.
Any agent that executes steps will report the steps as done. Whether the *outcome* exists in the
world is a separate question that requires separate eyes.

---

## Part 2 — The incident log (what actually went wrong, verbatim)

Every lesson in this playbook was paid for. These are the receipts.

### Incident 1: The blank tax form (the case for the supervisor)

A US marketplace launch required a W-8BEN (the IRS form that prevents 30% withholding for non-US
founders). The builder filled the official PDF and returned a detailed success report — field
mapping tables, per-line confirmations, "ready to use."

The supervisor's standing rule — *render every document to pixels before it leaves the machine* —
converted the PDF to an image: **completely blank, and structurally corrupted** (broken xref
table, invalid field references; the PDF used XFA forms, which the fill library silently mangled).

The rebuild used a different, verifiable method: stamp the values onto the untouched original as a
flattened text overlay, render, eyeball every field, then upload. Total detection cost: one render
and one look. Cost if missed: 30% of every payout, silently.

**Rules extracted:** (a) self-reported success is worthless; (b) render documents before
submission, always; (c) when a form-fill library meets XFA, prefer overlay + flatten.

### Incident 2: 781 views, 0 trials (the case for stranger-walking)

For weeks, "make sure the site is ready" produced reassuring audits. The prompt "walk the site as
a skeptical stranger about to pay" found — in one pass — that the free trial demanded a credit
card at checkout, and that **781 landing views had converted to exactly 0 trials**. The wall was
invisible to every logged-in, card-on-file, builder-machine test.

**Rules extracted:** (a) prompts need failure conditions a machine can hit; (b) the stranger path
(logged out, no card, cheap device, first click) is walked before any "done"; (c) funnel metrics
are read next to the funnel, not in aggregate.

### Incident 3: The confident swarm vs. one real phone

A GPU inference bug got a 22-agent research fan-out. The agents converged unanimously on a
solution. The resulting build **crashed on native library load on the actual device.** Separately,
a single adversarial agent — brief: "here is finished, verified work; break it" — found **11 real
security/correctness issues** in a system the builders had verified green, including an approval
channel that trusted anyone who guessed a topic name.

**Rules extracted:** (a) consensus among AIs is not evidence — they share blind spots and your
framing; (b) one device test outranks any number of agreeing agents; (c) every verification pass
gets an adversarial stance ("refute this"), never a confirmatory one ("check this").

### Incident 4: The silently stripped accents

A game's Brazilian-Portuguese script shipped with **7 accented characters** (Spanish: 4,855;
French: 7,147) — a batch pipeline had stripped every diacritic. To the #1 install country the game
read as broken machine translation. The store rating (2.5★) found the bug before any test did.

**Rules extracted:** (a) localized output gets a structural diff (character-class counts catch
this in one shell line); (b) one native reader per language before ship; (c) "looks right to me"
carries zero information in a language you can't read — which for an AI includes *all* of them,
because it reads without noticing what a native notices.

### Incident 5: $0 across 1,050 installs

The same game's IAP was priced in auto-converted USD because regional pricing was "later."
The install base was LatAm-heavy. Revenue: **zero buyers, ever.** Nothing in the codebase was
wrong; the money path had simply never been walked for the actual audience.

**Rules extracted:** (a) the pricing path is a feature with tests; (b) price for the geography
you're acquiring, not the one you imagined; (c) "later" on monetization is a decision with a date,
or it silently becomes "never."

### Incident 6: The missed payout tab

A marketplace launch was declared complete — listing live, webhooks validated, redemption flow
tested. The **Billing tab sat unclicked** in the same nav bar; the payout onboarding (payment
processor, bank details, tax forms) was unfilled until the founder found it himself and asked,
pointedly: *"why didn't you flag this before?"*

The response became the system's favorite artifact: the supervisor wrote itself a permanent rule —
**trace one imaginary sale end-to-end to the founder's bank account before calling any launch
done** — and filed it in persistent memory, where every future session loads it.

**Rules extracted:** (a) launch ≠ live; launch = money can physically arrive; (b) enumerate every
tab of a partner dashboard once, systematically; (c) corrections become rules, or they repeat.

---

## Part 3 — The memory system: how corrections become law

Chat context dies. Anything you say in a session — corrections, preferences, hard-won gotchas —
dies with it unless externalized. The fix is a **persistent memory directory** the model reads at
the start of every session: one file per fact, one-line index, and a discipline of writing to it
the moment a correction lands.

Standing rules that emerged (each one was a repeated mistake first):

1. **Never deploy, push, or publish without an explicit instruction for that action, that turn.**
   "Commit this" is not "push this." The blast radius of publishing is unbounded; local actions
   are free to undo.
2. **Only claim what you have personally seen.** Recalled facts, subagent reports, and cached
   beliefs are hypotheses. Verify against the live system before asserting.
3. **Never state facts from memory — verify everything first.** Anything that can have changed
   since it was learned gets checked against a live source *before* it is asserted: library
   versions, API shapes, prices, model names, platform limits, current events — web-search them;
   the state of your own codebase — read the actual file; a config or flag recalled from a past
   session — confirm it still exists. Training data and session memory are recall, not knowledge,
   and both go stale silently. The corollary: cite the source you checked, so the claim is
   auditable. (This rule exists because confident stale answers — an old character limit, a
   renamed API, a "known" file that had moved — each cost a debugging detour before the rule
   made them impossible.)
4. **No AI attribution on public artifacts** (commits, PRs) — the founder's name is the identity.
5. **Money and identity questions get asked, not guessed** — bank rails, tax residency, legal
   names. Everything else: execute without confirmation.
6. **Walk the payout path** (from incident 6).
7. **Assume every repo goes public eventually** — secrets live outside the repo, always.
8. **Mobile audits assert the viewport** (exactly 390px, full-page screenshots, viewport≠device-width
   is a failure, not a footnote) — because "0 overflow ✅" once shipped a broken layout.

The meta-rule: anything said twice gets written down. The memory file records the *why* and the
*how to apply*, not just the *what* — future sessions need the reasoning to apply it in new shapes.

---

## Part 4 — The verification protocol (pixels, not confidence)

"Verified ✅" from a language model is a mood. Evidence is:

- **Documents:** rendered to an image and read, before leaving the machine (incident 1)
- **Web changes:** `curl` of the *production* URL, logged out, cache-busted, after every deploy —
  "deploy succeeded" is a claim about infrastructure, not about your feature
- **UI:** screenshots at the real viewport; numbers on screen get *read*, not asserted — a 110%
  return once displayed as "1.1%" through fully green tests
- **Uploads:** check the byte size landed (a sandboxed browser once attached a 0-byte file with no
  error, from a directory it couldn't read)
- **Data claims:** counted at the source (a "same-content" claim between two repo clones is
  checked with `git log HEAD..origin/main`, not eyeballed)

The asymmetry that justifies all of it: verification costs seconds; the failures it catches cost
weeks, ratings, or revenue percentages.

---

## Part 5 — Prompt patterns that changed outcomes (before → after)

| Before (produced nothing) | After (produced results) | Why it works |
|---|---|---|
| "Make sure the site is ready" | "Walk the site as a skeptical stranger about to pay; screenshot each step" | Failure becomes detectable |
| "Fix the bug" | "Reproduce it first; show me the failing state; then fix; then show the same state passing" | Proof brackets the change |
| "Check this work" | "Assume this is broken. Try to refute it. Default to 'broken' if uncertain" | Adversarial stance finds what confirmation can't |
| "Test on mobile" | "Assert innerWidth==390, full-page screenshot every data section, treat viewport≠device-width as a failure" | Removes the model's ability to grade its own homework |
| "Add localization" | "After translating, print accented-character counts per language file and diff against source" | Structural check catches silent fidelity loss |
| "What's the current limit/version/price of X?" (answered from training memory) | "Search or fetch the live source first, then answer, citing what you checked" | Recall goes stale silently; a live source is dated evidence |
| "Deploy it" | (Only accepted with an explicit go-word, that turn, followed by a logged-out prod smoke test) | Irreversibility gets a human gate + evidence |

And the highest-level pattern: **state intent, not steps** — but only after the rails exist.
Three-word prompts ("go", "do it all") work *because* months of standing rules constrain what
"go" can possibly do. Trust is earned one written rule at a time; it lives in the system, not
the model.

---

## Part 6 — The domain lessons (the other 90% of the pain)

Nine deep-dive files distilled from the same five months, each incidents-first:

| File | One-line takeaway |
|---|---|
| [lessons/01-verify-like-a-stranger.md](lessons/01-verify-like-a-stranger.md) | Every product broke on the first real user; five minutes of stranger-walking catches most of it |
| [lessons/02-money-and-gates-first.md](lessons/02-money-and-gates-first.md) | Pricing paths and abuse gates are features with tests, not afterthoughts — and walk the *payout* path too |
| [lessons/03-scope-discipline.md](lessons/03-scope-discipline.md) | Agents make scope creep free; milestone contracts and close-or-commit are the defense |
| [lessons/04-browser-automation-playbook.md](lessons/04-browser-automation-playbook.md) | Raw CDP beats frameworks for attached sessions; snap browsers, srcset, and 0-byte uploads will lie to you |
| [lessons/05-deploy-and-ops.md](lessons/05-deploy-and-ops.md) | push ≠ deploy ≠ working; caching eats fixes; stale clones are minefields |
| [lessons/06-seo-programmatic-playbook.md](lessons/06-seo-programmatic-playbook.md) | 20k programmatic pages taught: soft-404s, robots catch-22s, and comparisons-convert |
| [lessons/07-on-device-ai.md](lessons/07-on-device-ai.md) | "Unsupported hardware" usually means "nobody shipped the artifact" — 31.3 tok/s on a phone forums wrote off |
| [lessons/08-localization.md](lessons/08-localization.md) | Automation silently destroys fidelity; geography picks languages; localization includes money |
| [lessons/09-multi-agent-patterns.md](lessons/09-multi-agent-patterns.md) | Fan-outs need ground truth; verification is a separate stage with an adversarial stance |

---

## The one-line version

Prompting isn't the skill. The skill is building a **system** around the model — a builder that
ships, a supervisor that distrusts, rules that persist across sessions, and evidence requirements
that make "it works" mean something. The models keep getting smarter. The system is what turns
smart into shipped.
