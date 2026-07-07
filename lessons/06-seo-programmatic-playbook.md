# 06 — Programmatic SEO that actually gets indexed

FilingLens grew from a handful of pages to a 20,000+ URL programmatic surface
(per-stock pages, per-metric history pages, X-vs-Y comparisons, screener landings). Most of the
work wasn't generating pages — it was removing the traps that stopped them from indexing.

## Traps found in production (each one silently blocked indexing)

- **Soft 404s:** the catch-all route served the homepage with HTTP 200 for any unknown URL. To a
  crawler that means infinite duplicate pages and wasted crawl budget. Real 404s fixed it.
- **The robots/noindex catch-22:** a page was `Disallow`ed in robots.txt *and* carried a `noindex`
  meta tag. Google can't see the noindex on a page it isn't allowed to crawl, so the URL lingered
  in limbo. To noindex a page you must *allow* crawling it.
- **SSR serving the wrong version:** the server-rendered route still returned the old design while
  users got the new one client-side — crawlers indexed markup no user saw. Crawl your own pages
  with curl and read what the bot reads.
- **Orphan pages:** comparison/screen pages existed only in the sitemap with no internal links.
  Sitemap presence ≠ discoverability; every money page needs an in-site path to it.
- **Structured-data pedantry matters:** VideoObject rejected over upload-date timezone format.
  Validate in the search console, not just by eyeballing JSON-LD.
- **Blocking auth pages in robots.txt** caused warnings on pages you actually want crawlable
  (login/register are conversion surfaces with real queries).

## What moved the needle

- **IndexNow after every deploy:** one script pings the full URL set to Bing/DDG/AI-engine
  infrastructure instantly. Observed result: Bing indexed the programmatic pages **ahead of
  Google**. Google has no equivalent; for everything else, IndexNow is free reach.
- **Comparison pages convert crawl into clicks:** in a young site's Search Console, virtually all
  clicks came from X-vs-Y competitor/stock comparison pages — high-intent queries with weak
  competition. If you can only build one programmatic page type, build comparisons.
- **Canonical + OG hygiene on every generated page**, one `<title>`/`<h1>` pattern per page type,
  and gzip on the SSR output — boring, cumulative, worth it.
- **Sitemap sanity:** auto-regenerate, ping on deploy, and keep it truthful — a sitemap that claims
  URLs that 404 or redirect burns crawler trust.

## Process rules

1. Treat the search console as a bug tracker: every coverage warning is a defect with a root cause,
   not weather. (But verify its claims — a "stale error" on a provably valid sitemap happens; fetch
   the resource yourself before rebuilding anything.)
2. After every SEO-affecting deploy: curl the page as Googlebot, check status code, canonical,
   robots meta, and that the content is server-rendered.
3. Measure per-page-type, not sitewide. One winning page type (comparisons) was invisible inside
   a sitewide 0.3% CTR until the data was split.
