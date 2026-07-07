# 05 — Deploy and ops: the gap between "pushed" and "live"

## push ≠ deploy ≠ working

The costliest ops assumption: that `git push` means users have the change. On this stack the
hosting provider's git integration silently broke; pushes did nothing until deploys were triggered
explicitly via the provider's API. The rule generalizes:

1. **Verify the deploy pipeline exists before trusting it.** After any deploy, poll the deploy
   status to "live", then **smoke-test the actual production URL** — logged out, cache-busted —
   for the specific thing you changed. "Deploy succeeded" is a claim about infrastructure, not
   about your feature.
2. **Never deploy, push, or publish without an explicit instruction for that action, that turn.**
   "Commit this" is not "push this." "Save this" is not "deploy this." The asymmetry is total: a
   local commit is free to undo; a deploy/publish is visible, cached, and indexed.
3. **Assume repo visibility can change.** If deploys, CI, or integrations ever require flipping a
   repo public — even briefly — then secrets, personal data, and business-sensitive files must
   never be committed. Keep them git-ignored and verify with `git status --ignored` before any
   visibility change. Credentials live outside the repo entirely.

## The stale-clone trap

With multiple clones (or a user who ships from another machine), a local repo silently falls behind.
Worst case observed: two weeks stale, with local commits that were same-content-different-SHA
duplicates of what was already on origin — a rebase minefield. **Always `git fetch origin && git
log HEAD..origin/main` before starting work in a repo you didn't touch five minutes ago.**

## Caching will eat your fix

- Assets shipped with long TTLs mean a deployed fix isn't what users load. Cache-bust (version
  query, filename hash) anything you need users to get *now*.
- In-process caches without bounds became a production OOM. Every cache gets a max size the day
  it's born.
- Tiered TTLs by data volatility (60s quotes / 12h fundamentals / 24h news) kept a free-API-backed
  product fast and under rate limits — distinguish live data from stable data instead of picking
  one TTL.

## Resilience basics that paid off

- Retry external connections (DB, upstream APIs) with capped backoff instead of dying on boot.
- Decouple long-running work from requests (build async, poll for status) — the alternative was a
  flagship feature hanging for 5+ minutes on first user contact.
- Rate-limit every public POST endpoint the day it ships; production bots found the signup form
  within days.

## Environment gotchas worth keeping

- systemd user services can squat ports you assume are free — check `lsof -i:PORT` before blaming
  your own code.
- Background daemons your session started (debug browsers, dev servers) outlive the session; kill
  by port PID, not by name pattern (see the `pkill -f` self-match trap in 04).
