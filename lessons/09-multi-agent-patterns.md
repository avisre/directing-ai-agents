# 09 — Multi-agent patterns: fan-outs, adversarial review, and ground truth

Multi-agent orchestration was used for retrospection, hardening, research, and content pipelines.
What separates the wins from the waste is always the same thing: **whether the swarm's output is
anchored to ground truth an agent can't argue with.**

## Patterns that earned their cost

- **Mining fan-out with verification stage.** The portfolio retrospective ran 21 agents over
  ~3.9M tokens of chat history: 16 chunk-extractors → 4 per-project synthesizers → 1 cross-project
  synthesis. The load-bearing design choice: synthesizers **cross-checked every claim against git**
  and tagged anything unconfirmable as `[UNVERIFIED — no git]`. The tags turned out to matter more
  than the prose — they mark exactly where memory is storytelling.
- **Adversarial review as a distinct pass.** A review whose only brief was "break this" found
  **11 real security/correctness issues** in work the building agents had verified green —
  including a trust-the-broadcast-topic hole in an approval channel. Builders verify the feature
  works; only adversaries verify it can't be abused. Different prompt, different findings.
- **Parallel mechanical hardening.** 8 agents each owning one bounded checklist item (log
  migration, manifest audit, accessibility, dark mode) for a store-compliance pass. Works because
  each task is independently verifiable and no agent needs another's context.
- **Batch content pipelines with human override lanes.** 381 images / 72-scene translations run
  agent-driven with retry + guardrail detection, plus a manual override list for the items
  automation kept failing (9 of 381). Design for the tail: track per-item state, make the run
  resumable, expect to finish some by hand.

## The cautionary tale

A **22-agent research swarm** investigated a GPU inference failure and converged confidently on a
solution (an alternative runtime). The resulting APK **crashed on native library load** on the
actual device. Twenty-two agreeing agents were outranked by one physical test. Research swarms
produce *hypotheses*; only execution on the real system produces *answers*. Never let convergence
substitute for a device test.

## Rules

1. **Give every swarm a ground truth** — git history, a compiler, a device, a live URL. A fan-out
   whose outputs can't be checked against something non-negotiable produces confident fiction at
   scale.
2. **Make verification a separate stage with a separate stance.** Same-agent self-review passes
   its own work. Extraction → synthesis → adversarial check, with the checker prompted to refute.
3. **Mark unverified claims in the artifact itself.** `[UNVERIFIED]` tags are cheap when written
   and priceless when read weeks later.
4. **Parallelize only what's independent.** Mechanical checklists parallelize perfectly; design
   decisions don't — they need one context holding the trade-offs.
5. **Persistent memory needs the same discipline:** date-stamp state claims, record the *why* with
   the *what*, and treat recalled memories as point-in-time observations to re-verify — the file,
   flag, or behavior may have changed since it was written.
6. **The orchestrator supervises; it doesn't have to believe.** Read subagent results as claims to
   spot-check (does the cited file exist? does the test actually run?), not as facts to relay.
