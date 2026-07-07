# 04 — Browser automation playbook (what actually works in 2026)

Hard-won mechanics from driving real logged-in browsers for store listings, profile scraping,
form-driving, screenshot pipelines, and video rendering. This is the area with the highest gap
between "should work" and "works."

## Driving a real, logged-in browser

- **Chromium 136+ refuses remote debugging on the default profile.** Launch with a dedicated
  `--user-data-dir` (for snap browsers, somewhere snap-writable like `~/snap/<browser>/common/`).
- **Raw CDP over websocket beats Playwright's `connectOverCDP` for attached sessions.** Playwright
  hangs indefinitely on pages with certain service/blob workers (LinkedIn reliably kills it).
  A ~40-line python driver — `websocket.create_connection` on `webSocketDebuggerUrl`, then
  `Runtime.evaluate` / `Page.navigate` / `Page.captureScreenshot` — has no such dependency and
  never hangs. Keep one as a reusable script.
- **`Page.captureScreenshot` beats `page.screenshot()`.** Playwright waits for visual stability, so
  an infinite CSS spinner = 30s timeout. Raw CDP captures instantly, animation or not.
- **Sandboxed shells kill process groups in surprising ways.** `pkill -f <pattern>` matches the
  shell running it (exit 144, killing your own compound command mid-heredoc). Kill by port owner
  (`lsof -ti:9222`) instead, and launch long-lived browsers with `setsid ... </dev/null &` from a
  script file, not inline.

## Scraping and anti-bot reality

- **Sites serve different assets to different fetchers.** Anonymous curl → HTTP 999 (LinkedIn).
  Rewriting a CDN URL to a bigger variant → token-signed 403. The reliable path: drive the real
  logged-in browser and **intercept network responses** — capture the bytes the page itself
  downloaded rather than re-requesting them.
- **`img.src` lies; check `currentSrc`.** An `<img>` with `src=...100x100` and `naturalWidth: 475`
  means srcset picked a different file than the attribute shows.
- **Cloudflare-walled dashboards:** automation with a copy of the user's real browser profile
  (cookies + fingerprint) succeeds where fresh automation profiles get challenged forever.
- **Flickering/react-rerendered controls:** clicking by selector races the re-render. Move the
  mouse first (wake the UI), then click by screen coordinates.
- **Batch pipelines through consumer AI tools hit CAPTCHA/rate walls.** Design for resume: track
  per-item progress, retry with overrides, expect to finish a tail of items by hand.

## File uploads into a snap-confined browser

`DOM.setFileInputFiles` (CDP) attaches files to `<input type=file>` without any picker UI — but a
snap-packaged browser cannot read outside the user's home directory. A file under `/tmp` attaches
as **0 bytes with no error**; the page just quietly keeps saying "upload a file." Check
`input.files[0].size` after attaching, and stage uploads under `$HOME` for snap browsers.

## Rendering video from a browser

Real-time screen capture of a headless page is a dead end (compositor-dependent; Wayland/rootless
X11 give black frames). The robust pipeline is **frame-stepped**: drive the page state, capture a
PNG per frame via CDP, then assemble with ffmpeg using an ffconcat list with per-frame durations
(`fps=30,format=yuv420p`, `libx264`, `-movflags +faststart`). It renders slower than real time but
produces perfect frames every time, on any machine, including 8K.

## Meta-rule

Budget browser automation at 3× your estimate and verify every step with a screenshot. The DOM you
reason about and the pixels a user sees diverge constantly, and only the pixels are true.
