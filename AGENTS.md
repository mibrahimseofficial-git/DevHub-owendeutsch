# Owen Deutsch Photography — SEO / Maintenance Project

## Client
Owen Deutsch & Claire Gillman — owendeutsch.com (Chicago, IL)
Wildlife & bird photography archive. Stack: WP Engine + Cloudflare + WP Rocket.
Custom PHP theme `owendeutsch` + Advanced Custom Fields (ACF), D3.js globe,
11-parameter custom bird search. 8,978 bird entries (cp_birds), 147 lodges, 14,122 URLs.

## Environments
- **Staging:** https://owendeutsc2stg.wpenginepowered.com/
- **Live:** https://owendeutsch.com/
- Both are public/reachable (no password wall on staging front-end as of Oct 2026).
- Staging is a faithful clone — it reproduces every bug found on live.

## MANDATORY WORKFLOW (client requirement)
**All tasks are done on STAGING first.** Work through the client document on staging,
then present staging to the client for sign-off. Only after the client gives the go-ahead
do we promote to LIVE:

- Changes **inside the theme** -> replace the theme on LIVE.
- Changes at the **database level** -> push staging to live.

Because the site runs WP Rocket + WP Engine Object/Page Cache + Cloudflare Edge,
every change must trigger a coordinated cache purge:
WP Rocket -> WP Engine -> Cloudflare Edge.

**Verification gotcha:** always cache-bust (`?cb=<random>`) and confirm `x-cache: MISS`
before reading a page. WP Rocket serves stale HTML on a cache HIT, which makes fixed
issues look unfixed (this caused a wrong Task 2 verdict once). Also verify in a real
browser, not only curl, when the question is "does the page show results".

## Source documents (repo: mibrahimseofficial-git/DevHub-owendeutsch, branch main)
- `Owen-Deutsch-SEO-Audit-Presentation-Final.pdf` — the 16-page client audit (31 tasks).
- `Owen-Deutsch-Website-Maintenance-Site-Health-Review.docx` — maintenance/site-health review (21 issues).

## Deliverable conventions
- Client-style documents: navy (#142A45) + orange (#D96B1E), Calibri, footer
  "Owen Deutsch Photography · ... | Page N", prioritised roadmap at the end.
- Generate .docx with python-docx; .pdf with pypdf/pdflatex as needed.

## Questionnaire workflow (how the user wants tasks handled)
One task at a time, in CLIENT DOCUMENT order. The client PDF (31 tasks) is completed
first; our DOCX (21 suggestions) waits for client go-ahead. For each task:
1. State the task as a question; give live-verified evidence + the verification method.
2. Agent verifies it, then STOPS and asks: real or false positive?
3. User verifies.
4. If real -> provide the solution (staging-first) and ask "should I apply it?".
5. User says yes -> apply it.
6. Both sides verify the result, then move to the next task.

## Verification notes / corrections to the PDF
- **Task 1 (8,424 broken location links):** FALSE POSITIVE as diagnosed. Site emits
  raw UTF-8 (en-dash E2 80 93), not %FF. `strtolower()` cannot produce %FF. The %FF is a
  crawler artifact from the DUAL charset declaration (UTF-8 + ISO-8859-1). Real fix =
  percent-encode the location query + remove the conflicting charset meta. Fix anyway
  because the client re-runs the crawler.
  - **CLOSED (staging).** Part A (%FF -> HTTP 400) verified fixed on staging: 0 `%FF` /
    0 `0xFF` bytes, legacy URL now 200. Part B (generated link returns 0 birds) declared
    a FALSE POSITIVE by the user. Note: a real-browser check showed "No birds found" for
    the site's own generated hyphen-form URL while the en-dash form returns 100 birds —
    an en-dash vs hyphen mismatch in the stored ACF data. Out of scope for Task 1; the
    audit's separate "normalize ACF filter values" task covers it.
- **Task 2 (birdingtop500 footer badge):** REAL, CLOSED (staging). The commented-out
  block has been removed on staging (verified: cache-busted raw refs = 0; live still
  shows 2 raw refs but 0 rendered). Badge server confirmed dead (HTTP 000, 20s timeout).
  Never rendered for visitors (was pre-commented in the client's original upload), so the
  "failed image request" symptom was a false positive; the cleanup is the real task.
  Repo `development` zip updated to match staging (block removed from footer.php).

## Known live-verified facts
- HTTP header sends `charset=UTF-8`. The legacy
  `<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">` has been
  REMOVED on staging (Task 1); LIVE still carries it.
- W3Schools placeholder keywords meta present on every page (staging + live) — OPEN.
- `%FF` appears 0 times in emitted HTML; raw 0xFF byte in query string -> 400.
- Correctly-encoded location URL returns 200 (full 1.7MB results page).
- No security headers (HSTS/X-Frame-Options/nosniff/CSP) on any page.
- debug.log, xmlrpc.php -> 403; REST user enum -> 401. (Good.)
- Plugin readme.txt files publicly readable (200).

## Branch workflow (client-directed)
- `main` = the client's uploaded theme snapshot (reference only, don't edit).
- `development` = **all changes go here.** The client downloads `owendeutsch.zip`
  from `development` and replaces the theme on STAGING, tests, then promotes to LIVE.
- Theme is stored in the repo as `owendeutsch.zip` (root). Working copy lives in
  `/tmp/theme/owendeutsch` after extraction; `/tmp/theme_backup_owendeutsch` holds
  the pre-change copy for diffs.
- Rebuild the zip with directory entries preserved (148 entries, top dir `owendeutsch/`).

## Client document task status
Work in CLIENT DOCUMENT order. Statuses: DONE (staging) / OPEN / FALSE POSITIVE.
Task numbering follows the user's "PDF#N" labels (31 tasks). Note the PDF's slide
"03 · 7 Relative Lodge URLs + 44 Dead External Links" is split into TWO client tasks:
PDF#3 = relative lodge URLs, PDF#4 = the 44 dead external links.

| # | Task | Status |
|---|------|--------|
| PDF#1 | 8,424 broken location links (HTTP 400) | DONE (staging) — Part B false positive |
| PDF#2 | Dead birdingtop500 footer badge | DONE (staging) — repo synced |
| PDF#3 | 7 relative lodge URLs missing scheme | DONE (staging) — commit `0558a7e` |
| PDF#4 | 44 dead external links | DEFERRED (user, 2026-11-02) — verified mostly real; ~13 client-named URLs broken, 6-7 now 200. Skip for now, revisit later |
| PDF#5 | W3Schools keywords meta | DONE (repo) — removed header.php L8; needs staging deploy |
| PDF#6 | Viewport meta blocks zoom (BUG 3) | DONE (repo) — header.php L9 simplified to width=device-width, initial-scale=1 |
| PDF#7 | Cache-Control/Pragma/Expires meta (BUG 4) | DONE (repo) — 3 tags removed from header.php. Note: did NOT actually defeat caching (x-cache HIT measured); removal is cosmetic/browser-side only |
| PDF#8 | Rendering failures: /cp_birds_country/ec/ + /lodges/southeast-california/ | DONE (repo) — audit claim (566 bytes, no <body>/title) is FALSE POSITIVE (both pages serve valid 200 HTML). Real defect fixed: country template emitted <script>+<style> AFTER get_footer() i.e. after </html>; moved both before get_footer(). Lodge page only trailing content is a WP Rocket comment (plugin, not ours) |
| PDF#9 | Robots.txt & search-parameter spider trap | PARTIAL (repo) — theme part DONE; two items are NOT theme-fixable. (a) Filter URL bloat: 51 emission sites across 6 templates now normalise ACF values via new `od_search_param()` (trim+mb_strtolower+rawurlencode), so "Flight"/" flight"/"FLIGHT" collapse to one canonical URL. Verified case/space variants already return identical result sets, so behaviour is unchanged. (b) `noindex, follow` on /?s= ALREADY present (WP core) — recommendation already satisfied. (c) `Crawl-delay: 10` in robots.txt is REAL but comes from Yoast/robots.txt, not the theme — CLIENT action. (d) /blog/page/2/ 301→/ is REAL but header shows `x-redirect-by: redirection` (Redirection plugin), not the theme — CLIENT action. (e) blog rel=next/prev is a FALSE POSITIVE — theme output is correct (page2 prev→/, next→page/3/) |
| PDF#10 | llms.txt & AI search discovery (slide 04) | DONE (repo) — `/llms.txt` was a genuine 404 on staging and live. Added `od_llms_txt()` on the existing `template_redirect` hook in functions.php: emits Markdown with a bio, both books, conservation/ABC, lodge directory, gear, FAQ, and all gallery links. Links built from `site_url()` so staging/live hosts resolve correctly. Sends `Content-Type: text/plain` + `X-Robots-Tag: noindex, follow` (machine input, not a landing page). All 14 referenced URLs verified 200 on staging. NOTE: implemented as a theme handler (not a physical file) because deploy is via owendeutsch.zip; a physical webroot llms.txt would work equally well if the client prefers. RESEARCH CAVEAT to surface to client: Google does NOT support llms.txt (Gary Illyes/John Mueller, confirmed; Google's 2026 AI-optimization guide lists it as a tactic to skip) and no major AI provider has committed to reading it. It is low-cost and harmless, helps agent/developer tooling, but must NOT be sold as a Google ranking or AI-Overview lever. |
| ... | remaining PDF tasks | not yet verified |

> **PUSH RESOLVED (2026-11-02):** `GITHUB_TOKEN` recovered; commits `4ecc5c1`
> (PDF#9) + `db17f79` pushed to `origin/development`. Branch in sync.
>
> **PDF#9 — robots.txt IS managed by Yoast** (confirmed from client screenshot:
> `Yoast SEO → Tools → File editor`). Live content matches the PDF exactly. Fix =
> delete the `Crawl-delay: 10` line in Yoast. RESEARCH CORRECTION: Google does NOT
> support `crawl-delay` (Google's own robots.txt docs list it as unsupported), so it
> does NOT throttle Googlebot — the audit's "also throttles Google" is wrong. It DOES
> throttle Bing, Yahoo (Slurp) and Yandex, which honour it. So the line is still worth
> removing, but it is a smaller issue than the audit implies.
>
> **PDF#9 — Redirection table reviewed (client export, 2026-11-02).** Two distinct groups:
> 1. BLOG PAGINATION rules `/blog/page/{2,4,6,10,11,13,16}/` → `/` (301). Many DUPLICATES
>    (page/6 ×3, page/16 ×3, page/11 ×2). Huge hit counts: one entry 45,478; also 138, 77,
>    35, 34, 13, 11. This is direct evidence of the crawl waste the audit describes — bots
>    repeatedly requesting pagination URLs that 301 away. NOTE page/3 is NOT in the rule
>    list, which matches our measurement that page/3 returns 200.
> 2. SECURITY-PROBE rules (legitimate hardening, NOT SEO-relevant): `/blog/wp-login.php`
>    (×3), `/wp-blog.php`, `/blog/wp-commentin.php`, `/blog/wp-admin/includes/`,
>    `/wp-content/themes/blogood/{img_screen,doc}.php`, `/blog/wp-content/themes/seotheme/mar.php`,
>    `/blog.php`, `/testing-blog/` → `/`. These redirect known bot/hacker probe paths away.
>    Keep them; optionally change 301 → 410 Gone (301 wrongly says "moved permanently").
>
> RECOMMENDATION (unchanged, SEO-best): DELETE the `/blog/page/N/` → `/` rules. Paginated
> pages are real content (page/2 = 200 at origin with real posts); a 301 to the homepage
> falsely says "this IS the homepage". Keep infinite scroll for humans. Then purge the
> Cloudflare edge cache (the 301 is cached, cf-cache-status: HIT). A wildcard rule is WORSE
> (hides more content; risks breaking YITH scroll, which fetches the next-page URL via AJAX).
> CLIENT ACTION: Tools → Redirection (delete pagination rules) + Cloudflare purge.
>
> **PDF#9 — VERIFY PASS 2026-11-02 (after client edits):**
> * Redirection list now **12 items** — all blog-pagination rules DELETED ✅. The 12 that
>   remain are security probes (`/blog/wp-login.php` ×3, `/wp-blog.php`, `/blog/wp-commentin.php`,
>   `/blog/wp-admin/includes/`, `blogood/*.php`, `seotheme/mar.php`, `/blog.php`, `/testing-blog/`)
>   plus one spam-comment URL. Correct to keep.
> * **LIVE robots.txt STILL shows `Crawl-delay: 10`** — verified with a cache-buster
>   (cf-cache-status: MISS), so it is NOT an edge-cache artifact; the live origin still
>   emits it. The Yoast edit either did not reach live or Yoast is serving a cached copy.
>   NEEDS FOLLOW-UP.
> * **STAGING robots.txt = `User-agent: *` / `Disallow: /`** — this is WP Engine's standard
>   staging lock, NOT the Yoast output. Staging overrides robots.txt entirely, so staging
>   cannot be used to verify the Yoast robots.txt change. Verify on LIVE only.
> * Staging `/blog/page/2/` = **200** ✅ and `/blog/page/4/` = **404** (only 3 pages exist) —
>   matches expectation.
> * ⚠️ **Cloudflare edge cache must be purged.** The old 301 for `/blog/page/2/` was served
>   with `cf-cache-status: HIT`; deleting the plugin rule does not evict the cached 301.
>   Until purged, `/blog/page/2/` keeps redirecting.
> * ⚠️ **BlogVault Firewall now blocks this container's IP on live** ("Blocked because of
>   Malicious Activities", reference ID) after a burst of automated requests. Live cannot be
>   verified from here right now; the client must verify in a normal browser. Note: this is a
>   WAF false-positive against an automated client, not evidence that real crawlers are blocked.
>
> **PDF#9 — STAGING RE-VERIFY 2026-11-02 (client confirmed: all work is on STAGING):**
> * Redirect deletion **VERIFIED WORKING on staging**: `/blog/page/2/` = **200**,
>   `/blog/page/3/` = **200**, `/blog/page/4/` = **404** (only 3 pages exist). Previously
>   staging served `301 location:/ x-redirect-by: redirection` for page/2 — now fixed. ✅
> * Staging page/2 has real posts ("Ibises of the World", "The Migrants Who Stay Close to
>   Home", "The Female of the Species"). ✅
> * **Staging is a fully locked environment**: robots.txt = `User-agent: * / Disallow: /`
>   AND site-wide `<meta name='robots' content='noindex, nofollow'>`. This is WP Engine's
>   staging lock, NOT Yoast output. **Consequence: the Yoast robots.txt change (removing
>   Crawl-delay) CANNOT be verified on staging** — the platform lock overrides it. It is
>   only observable on LIVE after deploy. Do not treat staging robots.txt as evidence.
> * Staging sits behind its own Cloudflare (`server: cloudflare`). Client purged cache via
>   **WP Rocket**.
> * WORKFLOW CLARIFIED: changes are made on STAGING (theme + Redirection rules + Yoast
>   settings live in the DB), then deployed staging → live. After deploying to live, purge
>   BOTH WP Rocket AND the Cloudflare edge cache — the old live 301 was served with
>   `cf-cache-status: HIT`, and a plugin-rule deletion does not evict a cached edge response.

## Our own additions (NOT in the client document)
Findings from our own crawl that the client PDF does NOT name. Tracked separately from
the 31 client tasks; surface to the client as extras, do not fold into a PDF task number.

| # | Finding | Impact | Status |
|---|---------|--------|--------|
| OA-1 | Zoom webinar registration 404 in the (commented-out) home popup, footer.php L122-151, plus orphaned trigger in js/custom.js L358-370 | URL string present on 453 pages, but never rendered (inside an HTML comment) — no user/SEO impact | OPEN — verdict pending |
| OA-2 | Typo'd self-link `https://www.owendeustch.com` on `/lodges/arizona-usa/` H1 (missing the "s"; edit distance 2 from `owendeutsch.com`, NXDOMAIN) | Headline lodge link points at a non-existent domain | OPEN — verdict pending |
| OA-3 | ABA (`aba.org`) 502 cluster — 6 URLs (root returns 200, sub-pages 502) | Likely a server-side outage at ABA, not our bug | OPEN — needs re-check before treating as ours |

Detection method for OA-2 (typo'd self-link): extract hrefs, compute edit distance of each
host against the canonical domain, and DNS-check the near-miss. Status code alone cannot
distinguish a self-link typo from an unrelated dead partner domain (distance 2 vs 12-14).

## External link crawl (baseline, Task PDF#4)
Crawled all 454 sitemap pages -> 348 unique external URLs (177 hosts). 3 retries each.
Result: 221 OK, 20x 404, 76x 403 (mostly bot-blocking: B&H affiliate, TripAdvisor,
AllAboutBirds, DOI), dead domain/server (000/502/503/522/530), 5x 429, 4x 400/401/402.
The PDF's "44" does not reconcile (likely counted 403 bot-blocks).
Raw data: /tmp/extall.json (links->pages), /tmp/extresall.json (status).

**Crawl false negatives (do NOT trust curl for liveness here).** Our crawler reported
several partner domains as dead that the user confirmed are LIVE and accessible:
`sachalodge.com`, `wildsumaco.com`. curl with a browser UA failed with PROTOCOL_ERROR /
timeout while a real browser loaded them fine. Treat curl-only "dead domain" results as
UNRELIABLE — confirm in a real browser before reporting a domain as dead. Do not
re-verify a domain the user has already confirmed live.

**Confirmed-dead domains (excluding the false negatives above):** hummingbirdscanada.ca,
beyourownbirder.com, femalebirdsong.org, africanpenguinsafe.org (530), cabanassanisidro.com,
dunesnationalpark.org, kws.go.ke, lemalacamp.com, lincolnparkconservancy.org, mendosa.com,
owletlodge.org (522), + a malformed DOI URL.

## Task 1 status (DONE — committed to development)
Commit `4240f99`. Added `od_location_search_url()` to functions.php (rawurlencode +
mb_strtolower). Replaced 15 raw location-URL sites across single-lodges.php,
single-cp_birds.php, search.php, grid_template.php, taxonomy-cp_birds_country.php,
templates/template-conservation.php. Encoded the 2 hardcoded lodge URLs. Removed the
ISO-8859-1 charset meta from header.php. All PHP passes `php -l`.
Note: `data-bird-locations` JS path is already safe (jQuery.param encodes).

## Task 2 status (DONE — repo synced with staging)
Removed the commented-out birdingtop500 block from `footer.php` (was lines 100-103)
in `owendeutsch.zip` on `development`. Staging already had this removed (edited directly
on staging, outside the repo); the repo now matches. Only `footer.php` changed
(148 zip entries preserved, `testzip()` clean).

## Task 3 status (PARTIAL — lodge URLs committed)
Commit `0558a7e`. Added `od_normalize_url()` (prepends `https://`, upgrades `http://`)
and applied it to the lodge title link + both `credit_url` outputs in `single-lodges.php`.
The external-broken-links half of Task 3 is still OPEN and needs a full crawl to
re-measure (the PDF's "44" could not be confirmed from a sample; several reported links
return 200).
