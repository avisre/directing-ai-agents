# Project by project: what each one taught

Six products, listed with what happened and the learnings extracted from each. Every claim here
comes from a real incident — most were reconstructed in a 21-agent retrospective over the full
build history (~3.9M tokens of session logs, 315 events, cross-checked against git).

---

## 1. FilingLens — AI stock research SaaS

**What it is:** Web app where every AI answer about a US stock is sourced from actual SEC filings,
cited, or refused. Free screener (3,800+ companies), 19 years of financials per company, guru
13F portfolios, filing-change monitoring. Node/Express + MongoDB + vanilla JS + D3. Now live on
the marketplace as a lifetime deal.

**Learnings:**

- **The funnel wall nobody saw:** a card-required trial converted 781 landing views into 0 trials.
  Caught only when the audit prompt changed from "make sure it's ready" to "walk it as a skeptical
  stranger about to pay." *Learning: prompts need failure conditions; strangers find what builders
  can't.*
- **The payout path is part of the launch:** the marketplace launch was "complete" (listing live,
  webhooks validated) while the billing/payout onboarding sat unfilled in an unclicked tab — and
  the tax form an AI later "filled" rendered completely blank until a supervisor model rendered it
  to pixels before upload. *Learning: trace one imaginary sale to the founder's bank account;
  render every document before submission.*
- **push ≠ deploy ≠ working:** the host's git integration silently broke; deploys had to be
  triggered via API and then smoke-tested logged-out on the live URL. *Learning: "deploy
  succeeded" is a claim about infrastructure, not your feature.*
- **Programmatic SEO is trap-removal, not page generation:** 20,000+ URLs indexed only after
  fixing soft-404s (catch-all served the homepage as 200), a robots/noindex catch-22 (can't read a
  noindex on a page you've disallowed), orphan pages, and SSR serving stale markup. Nearly all
  early clicks came from one page type: X-vs-Y comparisons. *Learning: crawl your own site as the
  bot; build comparison pages first.*
- **Data accuracy beats coverage:** Toyota's JPY financials got stamped as USD ($50.7T company!),
  so all foreign ADRs were cut from the AI's universe rather than risk one wrong number.
  *Learning: for a trust product, deleting bad data is a feature.*
- **Numbers on screen need eyes:** a 110% ROE displayed as "1.1%" through fully green tests.
  Long-running reports hung behind a spinner promising "~1 minute" for 5+ minutes on the first
  real user's test. *Learning: read the rendered page; decouple long work and show honest
  progress.*
- **Caches: bound them and bust them.** An unbounded in-memory cache OOM'd production; a 1-hour
  asset TTL meant deployed fixes weren't what users loaded. *Learning: every cache gets a max size
  and a busting strategy the day it's born.*

## 2. Localyze — offline AI assistant (Android + desktop)

**What it is:** Private, on-device LLM assistant — Gemma on phones via LiteRT-LM, native desktop
apps (WinUI 3 / SwiftUI / Qt6) with hardware-adaptive inference (NPU→dGPU→iGPU→CPU). 10 Indian
languages at 94%+ accuracy.

**Learnings:**

- **Launch-day latency nobody measured:** the free-tier model took **52.6 seconds to answer "Say
  hello"** (0.0 tok/s) on the device class free users actually own. Internal tests all ran on
  better hardware or the paid path. *Learning: test the worst supported device, on the free path,
  before ship.*
- **The paywall pointed at nothing:** the paid products didn't exist in the store console at
  launch, so free users were locked out of a purchase that was impossible to make. *Learning: walk
  the money path end-to-end in production, not in theory.*
- **GPU delegates lie about memory:** an int8 model on disk (1.47GiB) de-quantized to FP16 at
  engine init (2.5–3GB) and OOM'd the GPU — the delegate was float-only. *Learning: disk size ≠
  runtime size; know your delegate's compute dtype.*
- **The CPU fallback masks every GPU failure:** the app "worked" while silently pinning a core.
  *Learning: log which backend actually ran; treat unexpected fallback as a bug, not a save.*
- **KV-cache is the lever:** cutting context 8192→2048 tokens made mid-range GPUs viable.
  Guardrails (fuzzy repetition-loop detector with mid-stream cancel, harness-leak post-processor)
  belong at one choke point, not 27 call sites.
- **Language scope explodes:** each added language multiplied testing, and per-language failures
  (Malayalam 60%→100% via retry fixes) each took dedicated cycles. *Learning: let acquisition
  geography pick languages; cap the set.*

## 3. The Ledger Report — narrative strategy game (1947 India)

**What it is:** Turn-based narrative game framed as a traditional ledger; 5 protagonists, hidden
Truth stat, permanent choices; Flutter; 11 languages; ~1,050 Play Store installs.

**Learnings:**

- **$0 across 1,050 installs:** the $2.99 IAP had zero buyers ever — the install base was
  LatAm-heavy and regional pricing was never set. The feature worked; the price was for a
  different country. *Learning: price for the geography you're actually acquiring; monetization
  deferred without a date becomes never.*
- **Accents stripped, rating destroyed:** batch localization left 7 accented characters in the
  Brazilian-Portuguese script (vs 4,855 in Spanish) — the #1 market read machine-translation
  gibberish and rated 2.5★. The fix (batch accent-restoration, verified lossless per scene, plus
  tutorial + localized listings) took ratings to 5★ on post-fix builds. *Learning: structural
  diffs (character-class counts) + one native reader per language; map every rating to its build
  and language cohort to prove causality.*
- **Choices that don't matter get found out:** all 50 non-final scenes had identical outcomes
  regardless of choice — a disguised linear rail; historical events fired in clumps that
  steamrolled player agency (a −18 event on a character starting with 15 capital). *Learning:
  systems get audited by players even when tests pass; simulate the actual experience.*
- **Close or commit:** a full branching rework (stat-gated endings, debt-interest system, 5-way
  convergence finale) was built and verified in an emulator — and never integrated into the live
  build. *Learning: "locally working, ship in limbo" is the worst state; integrate or shelve
  explicitly.*
- **Grandfather your early users in code:** the freemium switch auto-granted the unlock to
  existing players via a migration flag that survives data wipes. *Learning: loyalty logic is
  cheap before the switch and impossible after.*

## 4. Discover Hinduism (Divya) — prayer & lore app

**What it is:** Kotlin Multiplatform app for the Hindu diaspora — 34 deity sections, 117 chapters
fully authored in 4 locales (EN/HI/TA/ML), 381 AI-generated illustrations. Live on Google Play.

**Learnings:**

- **Batch content pipelines need a manual override lane:** 381 images generated via an automated
  pipeline with guardrail detection and retries — and a hand-managed override list for the ~9
  subjects automation kept refusing. *Learning: design for the tail; track per-item state; expect
  to finish some by hand.*
- **Multi-locale from day 1 is cheaper than retrofit:** every screen shipped with localized nav
  and content variants from the start — the opposite order (Ledger Report) cost a rating disaster.
- **Store review is a compliance product:** a prior version of the app was suspended; the
  replacement passed by treating privacy policy, data-safety forms, and a support contact
  decoupled from personal email as first-class deliverables. *Learning: the listing is part of the
  product.*
- **Content as data:** chapters and deity lore live as data, not code — authoring scaled without
  touching the app. Shared KMM logic + platform-native UI avoided the cross-platform uncanny
  valley.

## 5. Snapdragon NPU LLM — research recipe

**What it is:** Reproducible recipe for running LLMs on the Hexagon NPU of "unsupported"
Snapdragon 8 Gen 1 phones — measured **31.3 tok/s, 107ms TTFT** on a OnePlus 10 Pro.

**Learnings:**

- **"Unsupported" usually means "nobody shipped the artifact":** forums, docs, and tutorials all
  said 8 Gen 1 couldn't run NPU LLMs. The runtime had supported it in source for a year; what was
  missing was one published compiled model file. When one appeared (3 downloads, no announcement),
  it unblocked the entire device generation. *Learning: separate hardware capability from artifact
  availability before accepting "can't."*
- **Benchmark before architecting:** on this silicon, INT8 on the NPU measured **0.42 tok/s** —
  versus 7–10 tok/s on the GPU. "Runs on NPU" and "faster on NPU" are different claims.
- **Quantization pipelines fail in mundane ways:** rejected ops, host-RAM OOM during conversion,
  dtype gaps, buffer-name collisions. *Learning: budget for the pipeline, not just the model.*

## 6. Control Tower / AI Avatar — agent-cost monitor (prototype)

**What it is:** Desktop companion that renders a VRM avatar and tracks what AI agents cost in real
time — per-agent attribution, budget runway, spend gates with phone approval. Electron first, then
a Rust/Bevy rewrite (1.7MB binary vs ~200MB).

**Learnings:**

- **Adversarial review beats builder confidence:** a "break this" pass on verified-green work
  found **11 real security/correctness issues**, including an approval channel that trusted anyone
  who guessed a public topic name. *Learning: every feature that moves money or approves actions
  gets a hostile review before exposure.*
- **The 22-agent research swarm was wrong:** it converged confidently on a GPU fix; the build
  crashed on the real device. *Learning: consensus is not evidence; one device test outranks any
  number of agreeing agents.*
- **Performance bugs hide behind "works":** the avatar defaulted to software rendering and pinned
  a CPU core silently; a budget gate re-read the whole transcript per tool call (1.85s) until an
  append-only offset cache cut it to 37ms. *Learning: measure, don't vibe; append-only data wants
  offset caches.*
- **Scope creep is the product killer:** Phase 1 desktop app → STT/TTS → Windows CI → Rust port →
  Bevy VRM renderer → a second mascot port. Five products of scope, zero shipped to a user.
  *Learning: define per-milestone success up front; ship, then decide expansion.*
- **Be honest about the market:** the competitive review concluded the avatar caps mainstream
  appeal for a serious monitoring tool — recorded as a finding, not buried. *Learning: an honest
  "this probably won't win" is a deliverable too.*

---

## The cross-project pattern

Four traps recurred on every single project, independently:

1. **Ships-broken:** features work when demonstrated, break on the first real user (Localyze
   52.6s hello; FilingLens 0-trials wall; Ledger's 5-minute hang).
2. **Gates-last:** rate limits, quotas, and abuse paths bolted on after launch (or after the
   adversarial review finds the holes).
3. **Monetization-late:** the pricing layer stays a stub until traffic proves demand — and then
   the gap is discovered in lost revenue ($0 across 1,050 installs).
4. **Scope-creep:** each increment ships before the previous one is hardened, because "we're
   already here."

The system that counters them — builder/supervisor model split, standing rules, stranger-walks,
evidence requirements — is documented in the [README](README.md) and the
[lessons/](lessons/) files.
