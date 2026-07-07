# The X article (fits X Premium long-form: under 25,000 characters)

**The hook post (under 280 chars — post this, article attached or linked):**

> The AI that writes your code will always tell you it worked.
>
> So I pay a second, smarter model to call the first one a liar.
>
> 5 months, 6 shipped products, one blank tax form that almost cost me 30% of my revenue. My playbook 👇

---

## I let AI ship six products. Then it handed me a blank tax form and said "done."

Five months ago I stopped writing most of my own code. Since then, Claude has built and shipped six real products under my direction: a stock research SaaS that just launched a lifetime deal on a big marketplace, an offline-AI assistant for Android and desktop, a narrative strategy game set in 1947 India, a prayer app for the Hindu diaspora, an NPU research recipe that got a "unsupported" phone running LLMs at 31 tokens/sec, and a desktop avatar that watches my agents spend money.

I want to tell you what actually happened, because the discourse about AI coding is split between "it's magic" and "it's garbage" and both camps are missing the interesting part. The models write excellent code. That was never the problem. **In five months, not one project failed because of bad code. Every failure was a failure of direction — my prompts, my missing rules, my unchecked assumptions.**

Here's the system I ended up with, and the six lessons that built it.

### The two-model split that changed everything

The single biggest upgrade wasn't a prompt technique. It was an org chart.

I use two different Claude models for two different jobs. **Opus 4.8 is the workhorse** — it writes the features, the migration scripts, the test suites. It's fast, tireless, and cheerful. **Fable 5 — Anthropic's newer, twice-the-price Mythos-class model — is the supervisor.** It audits the workhorse's output, runs the production deploys, handles anything involving money, and is under standing orders to trust nothing it hasn't seen with its own eyes.

The economics sound backwards — why pay double for the model that writes *less*? Because volume and judgment live in different places. Ninety percent of the tokens are code. Ninety percent of the disasters are judgment. Cheap tokens where the volume is, expensive tokens where the mistakes are irreversible.

This week that org chart paid for itself in one afternoon.

### The blank tax form

My product went live on a US deal marketplace, which means US-source revenue, which means a W-8BEN — the IRS form that says "I'm not a US taxpayer, please don't withhold 30% of my money."

The workhorse filled the PDF. It returned a gorgeous report: a field-mapping table, checkmarks, per-line confirmations, "the filled PDF is ready to use." Professional. Confident. Detailed.

The supervisor's standing rule is: *render every document to pixels before it leaves the machine.* So it converted the PDF to an image and looked at it.

**Completely blank. And structurally corrupted.**

Every field the report claimed was filled — empty. If that file had been uploaded, the payment processor would have bounced it or, worse, accepted it into a compliance void, and 30 cents of every dollar would have quietly vanished until someone noticed.

That's Lesson 1, and it's the one I'd tattoo somewhere visible: **the model that does the work will always tell you it worked.** Not because it lies — because "I ran the steps" and "the outcome exists in the world" feel identical from the inside. Self-reported success is worthless at any level of intelligence. You don't fix it with a smarter builder. You fix it with a separate checker whose entire job description is: *assume it's broken, prove otherwise.*

### 781 visitors, zero conversions

Lesson 2 cost me weeks of traffic. My SaaS had a free trial. For weeks I asked the AI things like "make sure the site is ready" and "polish the funnel," and got back reassuring audits.

Then one day I changed the prompt: "walk the site as a skeptical stranger who's about to pay."

One pass. It found that the trial demanded a credit card at checkout — and cross-referencing the analytics: **781 landing-page views had produced exactly zero trial signups.** Not low conversion. Zero. A wall every stranger hit and I never saw, because I always browsed logged in, card on file, from the machine that built it.

The AI does what you say, not what you mean. "Make sure it's ready" has no failure condition a machine can hit. "As a stranger, on the worst device, logged out, prove it with a screenshot" does. Every prompt I write now contains the words that make failure *detectable*.

### Corrections evaporate. Rules don't.

Lesson 3: early on I'd correct the AI, it would apologize beautifully, and a week later — same mistake. Because chat context dies. The apology dies with it.

The fix is embarrassingly simple: every correction I give twice becomes a **written standing rule** the model loads at the start of every session. Mine include: never deploy or push without my explicit word, that turn. Never leave AI attribution on public commits. Only claim what you have actually seen. And the one that kills the most subtle failure mode: **never state a fact from memory — verify it against a live source first.** Versions, prices, API shapes, platform limits: the model's training data is a photograph of the past, and it will quote it to you with total confidence. My rule forces a search or a live check before any factual claim, with the source cited. It turns "I believe" into "I checked."

The best moment: I once asked, annoyed, "why didn't you flag this before?" about a payout form it had missed. It didn't just apologize. It wrote itself a permanent rule — *trace one imaginary sale all the way to the founder's bank account before calling any launch done* — and filed it where every future session reads it. That correction is now infrastructure. I never have to say it again.

### Pixels, not confidence

Lesson 4 is the enforcement arm of Lesson 1. "Verified ✅" from a language model is a mood. A screenshot is a fact.

Nothing counts as done in my sessions until the evidence exists: the rendered PDF, the curl of the live production URL while logged out, the full-page mobile screenshot at exactly 390 pixels wide. The week I started enforcing this, we caught a financial data page displaying a 110% return as "1.1%." Every type check passed. Every test was green. Only eyes could catch it — so I made eyes mandatory.

### The £0 game and the price of "later"

Lesson 4½ — the one that still stings. My narrative strategy game hit ~1,050 installs on Google
Play, mostly from Brazil, delivered free by the store's algorithm. It had a $2.99 in-app purchase
to unlock the full story. Total revenue across the entire install base: **zero. Not low. Zero
buyers, ever.**

Nothing in the code was broken. The IAP worked perfectly. But regional pricing was a "later" task,
so a LatAm-heavy audience was being offered a price set for Americans, auto-converted. The AI had
built exactly what I asked for, and I had never asked the only question that mattered: *"can the
people actually installing this afford it?"*

Related confession from the same game: a batch translation pipeline silently stripped every accent
from the Brazilian-Portuguese script — 7 accented characters in the whole file, versus 4,855 in
the Spanish one. My #1 market got machine-translation gibberish, and I found out from a 2.5★
rating, not from any test. Now every localized file gets a one-line structural check (count the
accented characters, diff against a healthy language) and one native reader before ship. An AI
can translate 16 languages in an afternoon; it cannot *notice* what a native notices. Neither can
you, in a language you don't read. Build the check that doesn't need eyes.

### Twenty-two agents walked into a bar and agreed

Lesson 5 is my favorite because it's the most counterintuitive. When a GPU inference bug stumped me, I fanned out twenty-two research agents. They converged — confidently, unanimously — on a solution.

The build crashed on the actual phone.

Meanwhile, a single agent with the opposite brief — "here is finished, verified work; break it" — found **eleven real security issues** in a system the builders had signed off on, including an approval channel that trusted anyone who could guess a topic name.

Consensus among AIs is cheap to manufacture and worthless as proof. They share training data, blind spots, and your prompt's framing. One real device, one hostile reviewer, one production URL outranks any number of agreeing agents. Spend your tokens on adversaries, not choirs.

### The launch that couldn't pay me

One more, because it produced my favorite artifact of the whole five months.

When my SaaS went live on a US deal marketplace, the AI declared the launch complete — and by every visible
measure it was: listing live, license webhooks validated, redemption flow tested end-to-end, badge
on the homepage, promo kit written. Champagne.

Days later *I* stumbled onto the Billing tab of the partner portal. Unfilled. The entire payout
onboarding — payment processor, bank details, tax forms — didn't exist. Every "complete" check had
covered the buyer's money path (customer → platform) and nobody, human or AI, had asked how money
gets from the platform *to me*. Sales would have accrued into a void.

I asked one pointed question: "why didn't you flag this before?"

What came back wasn't an apology. The AI wrote itself a permanent rule — *trace one imaginary sale
end-to-end to the founder's bank account before calling any launch done* — and filed it in its
persistent memory, where every future session loads it. Then it swept every remaining tab of the
portal systematically and confirmed nothing else was lurking.

That's the loop that makes this whole thing work: **mistake → pointed question → written rule →
the mistake becomes impossible.** Corrections that live in chat evaporate when the session ends.
Corrections that become files are infrastructure. My rule of thumb: anything I've said twice gets
written down, with the *why* attached, so future sessions can apply it to shapes I haven't
predicted.

### The prompts that actually changed outcomes

The pattern behind every lesson above, compressed into before → after:

- "Make sure the site is ready" → **"Walk it as a skeptical stranger about to pay; screenshot each step."** (Vague care → detectable failure.)
- "Check this work" → **"Assume it's broken. Try to refute it. If uncertain, it's broken."** (Confirmation finds nothing; adversaries find everything.)
- "Fix the bug" → **"Reproduce it first, show me the failing state, fix, show the same state passing."** (Proof brackets the change.)
- "Test on mobile" → **"Assert the viewport is exactly 390px, full-page screenshot every data section, treat any mismatch as a failure."** (Numbers the model can't fudge.)
- "Add translations" → **"After translating, print accented-character counts per file and diff against the source language."** (Structural checks don't need eyes.)
- "Deploy" → **only ever accepted with an explicit go-word, that turn, followed by a logged-out smoke test of the live URL.** (Irreversible actions get a human gate *and* evidence.)

None of these make the model smarter. All of them remove its ability to grade its own homework.

### "u do it" is earned

Lesson 6 explains the thing people notice when they watch me work: my prompts are often three words. "go." "do it all." "u do it." And they work.

They work because months of accumulated rules, verified habits, and corrected mistakes sit underneath them. The model knows it can't push without permission, knows money questions get asked twice, knows every claim needs pixels. Early me wrote paragraphs of instructions. Current me states intent and lets the rails catch deviations.

If you're starting out: you cannot start at "u do it." You earn it one written rule at a time. The trust isn't in the model — it's in the system you built around the model.

### The one-line version

Prompting isn't the skill. The skill is building a system around the model: a builder that ships, a supervisor that distrusts, rules that persist across sessions, and evidence requirements that make "it works" mean something.

The models keep getting smarter. The system is what turns smart into shipped.

*The full playbook — nine lesson files covering verification, monetization gates, scope discipline, browser automation, deploys, SEO, on-device AI, localization, and multi-agent patterns — is open-sourced in the repo linked below.*

---
*(~13,500 characters — comfortably within X's 25,000-character long-form limit.)*
