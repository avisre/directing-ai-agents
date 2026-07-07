# 08 — Localization: silent fidelity loss and scope explosion

Localization work spanned a 16-language game, a 10-language on-device assistant, and a 4-locale
prayer app. Two distinct failure modes dominate.

## Failure mode 1: automation silently destroys fidelity

The canonical incident: the game's Brazilian-Portuguese story file shipped with **7 accented
characters** — versus 4,855 in the Spanish file and 7,147 in French. Somewhere in a batch
pipeline every diacritic was stripped. To Brazilian players (the **#1 install country**) the game
read as broken machine translation. The cost was direct and measurable: a **2.50★ rating** from
the store's biggest audience, discovered only post-ship via reviews.

The fix was equally instructive: batch accent-restoration with **per-file verification that the
change was accent-only and lossless** (72/72 scenes diffed), and post-fix ratings went 5★.

Rules:
1. **Diff localized output structurally.** Character-class counts (accented chars, script-specific
   glyphs) per language file catch stripped-diacritics-class bugs in one line of shell.
2. **One human native speaker reads each language before it ships.** Automated round-trips and
   LLM self-checks pass files a native reader rejects in seconds. Automation ≠ verification.
3. **Regional variants are not interchangeable** (pt-BR ≠ pt-PT). Target the variant your install
   base actually speaks.

## Failure mode 2: language scope explosion

Every new language multiplies every future change: 124 story scenes × each locale, screenshots ×
each locale, store listings × each locale, QA × each locale. Observed pattern on two products:
languages were added ahead of any demand signal, then each one generated its own bug tail
(retry loops to get Malayalam from 60%→100%, aspect-ratio reworks per locale's screenshots).

The moment of clarity came from checking acquisition data: **every top-5 country was already
covered** by shipped languages — the marginal language added cost with no reachable audience.

Rules:
1. **Let install/acquisition geography pick languages,** not completeness instinct. Ship top
   markets, then revisit with data.
2. **Localization includes money:** price localization (regional price tiers) is part of shipping
   a locale. A perfectly translated product with unaffordable USD pricing monetizes at zero
   (see 02 — this exact combination produced $0 across 1K installs).
3. **Localized store assets are part of the surface:** listings, screenshots, and the first-run
   tutorial carry as much rating weight as the app itself. The rating recovery came from fixing
   language + tutorial + listing together, and mapping every rating to build version + language
   to prove which cohort was unhappy.

## The general lesson

Localization bugs are invisible to whoever can't read the language — which includes the agent
doing the work. It is the one domain where "looks right to me" carries zero information, so the
process (structural diffs, native review, geography-driven scope) has to carry all the weight.
