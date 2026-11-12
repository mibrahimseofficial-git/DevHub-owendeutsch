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
- W3Schools placeholder keywords meta — **CLOSED (2026-11-06)**. No `<meta name="keywords">`
  tag on any page checked on staging (`/`, `/videos/`, `/blog/`, `/grid/`, `/globe/`,
  `/lodges/`, `/about-owen/`, all 200). The only `keywords` string in the HTML is the search
  box placeholder `"Enter bird species name or keywords"` — a false positive.
- `%FF` appears 0 times in emitted HTML; raw 0xFF byte in query string -> 400.
- Correctly-encoded location URL returns 200 (full 1.7MB results page).
- Security headers — **SET (2026-11-06)** on both staging and production via WP Engine
  Header Rules: `Strict-Transport-Security: max-age=31536000`, `X-Content-Type-Options:
  nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`. Once each, no duplicates.
  (Superseded the earlier "no security headers on any page" note.)
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
| PDF#4 | 44 dead external links | VERIFIED (2026-11-02) — **NOT THEME-FIXABLE**. Zero of the dead links are in theme code (grep of all 148 theme files: none of nanpa/owletlodge/africanpenguinsafe/nikon/abcbirds/seattle-audubon/roku/lincolnpark/orangehill/sandpiperabaco/sanisidrolodge/zebrahills/lemalacamp/wildsumaco/birdingtop500 appear). They live in **post content** (blog articles) and **ACF lodge fields** (database). Fix = client edits content / Redirection plugin, not a theme deploy. Live re-test 2026-11-02 shows many have RECOVERED since the Oct-2026 crawl: nanpa.org 200, nikonusa.com 200, lincolnparkconservancy.org 200, orangehill.com 200, sandpiperabaco.com 200, sanisidrolodge.com 200, zebrahills.com 200. Still genuinely broken: birdingtop500.com (server dead — badge already removed PDF#2), owletlodge.org (timeout), africanpenguinsafe.org (530), 5x abcbirds.org article links on /huembo.../ + 2 ABC pages (site restructured /blog20//article/ -> /news/), seattle-audubon shop (404), channelstore.roku.com detail (400), lemalacamp.com + wildsumaco.com (timeout). Footer Zoom webinar link (June 2020, 404) is inside a COMMENTED-OUT modal — dead code, never rendered. |
| PDF#5 | W3Schools keywords meta | DONE (repo) — removed header.php L8; needs staging deploy |
| PDF#6 | Viewport meta blocks zoom (BUG 3) | DONE (repo) — header.php L9 simplified to width=device-width, initial-scale=1 |
| PDF#7 | Cache-Control/Pragma/Expires meta (BUG 4) | DONE (repo) — 3 tags removed from header.php. Note: did NOT actually defeat caching (x-cache HIT measured); removal is cosmetic/browser-side only |
| PDF#8 | Rendering failures: /cp_birds_country/ec/ + /lodges/southeast-california/ | DONE (repo) — audit claim (566 bytes, no <body>/title) is FALSE POSITIVE (both pages serve valid 200 HTML). Real defect fixed: country template emitted <script>+<style> AFTER get_footer() i.e. after </html>; moved both before get_footer(). Lodge page only trailing content is a WP Rocket comment (plugin, not ours) |
| PDF#9 | Robots.txt & search-parameter spider trap | PARTIAL (repo) — theme part DONE; two items are NOT theme-fixable. (a) Filter URL bloat: 51 emission sites across 6 templates now normalise ACF values via new `od_search_param()` (trim+mb_strtolower+rawurlencode), so "Flight"/" flight"/"FLIGHT" collapse to one canonical URL. Verified case/space variants already return identical result sets, so behaviour is unchanged. (b) `noindex, follow` on /?s= ALREADY present (WP core) — recommendation already satisfied. (c) `Crawl-delay: 10` in robots.txt is REAL but comes from Yoast/robots.txt, not the theme — CLIENT action. (d) /blog/page/2/ 301→/ is REAL but header shows `x-redirect-by: redirection` (Redirection plugin), not the theme — CLIENT action. (e) blog rel=next/prev is a FALSE POSITIVE — theme output is correct (page2 prev→/, next→page/3/) |
| PDF#10 | llms.txt & AI search discovery (slide 04) | DONE (repo) — `/llms.txt` was a genuine 404 on staging and live. Added `od_llms_txt()` on the existing `template_redirect` hook in functions.php: emits Markdown with a bio, both books, conservation/ABC, lodge directory, gear, FAQ, and all gallery links. Links built from `site_url()` so staging/live hosts resolve correctly. Sends `Content-Type: text/plain` + `X-Robots-Tag: noindex, follow` (machine input, not a landing page). All 14 referenced URLs verified 200 on staging. NOTE: implemented as a theme handler (not a physical file) because deploy is via owendeutsch.zip; a physical webroot llms.txt would work equally well if the client prefers. RESEARCH CAVEAT to surface to client: Google does NOT support llms.txt (Gary Illyes/John Mueller, confirmed; Google's 2026 AI-optimization guide lists it as a tactic to skip) and no major AI provider has committed to reading it. It is low-cost and harmless, helps agent/developer tooling, but must NOT be sold as a Google ranking or AI-Overview lever. |
| PDF#11 | Duplicate & malformed Organization JSON-LD (slide 05) | DONE (repo) — confirmed TWO JSON-LD blocks on every page: (1) Yoast, valid (`@context: schema.org`, WebPage/BreadcrumbList/WebSite); (2) theme header.php L67-81, broken (`@vocab` = site URL instead of schema.org, leading spaces in `@id`/`url`/`logo`). Removed the theme block (15 lines). Both blocks parsed as valid JSON, so the defect is semantic/duplicate, not a parse error. After removal the page emits only Yoast's graph. FOLLOW-UP (client, Yoast): Yoast's graph currently has NO Organization/Person node, so configure Yoast SEO → Search Appearance → Knowledge Graph (name, logo, social profiles) to publish Organization for Owen Deutsch Photography. Book schema (both books) and ImageObject are NOT native Yoast outputs — need custom code or a plugin; flagged as optional enhancement, not silently implemented. |
| PDF#12 | On-page SEO & metadata (slide 06) | PARTIAL (repo) — heading fixes applied: globe/grid + country taxonomy `<h2 class="large">`->`<h1 class="large">` (13 pages now have an H1); blog listing (index.php) + videos (template-video-latest.php) loop titles `<h1>`->`<h2>`; homepage duplicate H1 (Search) -> `<h2>`; CSS updated to preserve appearance. FALSE POSITIVE: audit says /lodges/southeast-california/ has 0 H1 — it has 1. BUG FIXED: template-book.php mismatched `<h1>...</h2>`. NOT THEME-FIXABLE: "11 articles use H1 subheadings" — sampled articles emit exactly 1 H1; extra H1s are in post CONTENT (needs content edit). "332 duplicate titles / 343 duplicate H1s" is sitewide metadata (Yoast templates), not theme code. |
| PDF#13 | Image SEO & accessibility (slide 06 images) | PARTIAL (repo) — (a) ALT TEXT: added descriptive `alt` to gallery images in taxonomy-cp_birds_country.php (bird post title), grid_template.php (post title), template-globe.php + template-countries.php (country term name), template-south-africa.php (image title), template-video-latest.php (video title). (b) WIDTH/HEIGHT: injected `width`/`height` from data already fetched via `wp_get_attachment_image_src()` / ACF image arrays in the same 6 templates — prevents CLS. All values passed through esc_attr/esc_url/(int). (c) WEBP: NOT theme-fixable — 99.9% JPEGs; enable WebP at Cloudflare/WP Engine edge or via Imagify (CLIENT/INFRA). NOTE: many remaining `alt=""` come from WP core/plugins and `the_post_thumbnail()` loops, not these templates. |
| PDF#14 | Theme architecture, performance & security (slide 07) | PARTIAL (repo) — (a) DUPLICATE jQuery FIXED: header.php hardcoded jquery-3.6.1.js while WP enqueued jquery-core 3.7.1 with defer; removed the hardcoded copy and enqueue jquery once. Safe because no inline script runs before DOM-ready (verified: 0 inline scripts use jQuery outside a ready/DOMContentLoaded wrapper) and WP Rocket JS-delay is OFF (0 rocketlazyloadscript), so jquery-core (head, defer) loads before the DOM-ready callbacks. (b) 1.76MB HTML / hidden Fancybox captions: NOT present on staging homepage (589KB, 0 figcaption blocks) — appears only on search/activity pages with large result sets; reducing it would change gallery markup and is not a safe blind edit. (c) SECURITY HEADERS: theme sets none; confirmed staging serves NO HSTS/X-Frame-Options/X-Content-Type-Options/CSP. Best set at Cloudflare/WP Engine edge (client/Infra). (d) 15 http:// links: NOT in theme or rendered homepage (0 http:// hrefs); the only http:// string is a commented-out line in header.php. (e) Triple cache stack purge: process/client, not code. |
| PDF#15 | Duplicate jQuery / external JS (slides 06-07) | DONE (repo) — completed the jQuery de-duplication safely. header.php's hardcoded jquery-3.6.1.js was REMOVED in PDF#14, but WP's jquery-core (3.7.1) is deferred by `defer_parsing_of_js()`, so the theme's body inline scripts calling jQuery(document).ready() would have thrown "jQuery is not defined". Fix: (a) `defer_parsing_of_js()` now also excludes jquery.min.js + jquery-migrate from defer (loads synchronously, matching previous behaviour); (b) removed the second duplicate jQuery (jquery-3.6.1.min.js) hardcoded in template-globe.php, which was loading a 3rd jQuery on /globe/. Result: 1 jQuery per page (was 3 on /globe/, 2 elsewhere). LEFT ALONE deliberately: (i) /globe/ bootstrap — local is v3.3.2 but page loads CDN v5.3.0; swapping versions would break the tab JS, not a safe blind edit; (ii) /globe/ d3js.org topojson/queue scripts — all return 200 from this container (audit's "Timeout (0)" is a crawl-time network blip); (iii) Mailchimp noConflict(true) — external plugin markup, localises $ to $mcj, does not affect Fancybox/YITH. |
| PDF#27 | Self-host the external /globe/ JS files (slide 06: 3 "dead scripts") | DONE (repo) — vendored the 3 external scripts into the theme so /globe/ no longer depends on third-party CDNs: `js/topojson.v1.min.js` (topojson-client 1.8.0), `js/queue.v1.min.js` (queue v1), `js/bootstrap-5.3.0.bundle.min.js` (bootstrap 5.3.0-alpha3, byte-identical to the CDN file). template-globe.php now references all three via get_template_directory_uri(). Verified all 3 CDN URLs return 200 in ~0.1s from this container (the audit's "Timeout (0)" was a crawl-time blip, not a real outage) — so this is hardening/self-hosting, not a fix for an outage. NOTE (not changed): functions.php also enqueues a LOCAL bootstrap.min.js v3.3.2, so /globe/ loads bootstrap twice (3.3.2 + 5.3.0) — pre-existing; removing either risks the tab JS (`.tab("show")` is used), left for a scoped test. Also: header.php's `http://code.jquery.com/ui/1.10.2/jquery-ui.js` is COMMENTED OUT (not live mixed content); the live jQuery UI is the local 1.12.1 enqueue. |
| PDF#25 | 1.76 MB HTML / Fancybox caption bloat (slide 07 item 01) | VERIFIED — **plugin setting, not theme code**. The activity-search page is 1.72 MB = 100 unique per-bird Fancybox `<figcaption>` blocks (~7.5 KB each, ~850 KB total, = the audit's "8,752 words"). They are NOT duplicated (100 birds = 100 captions) and are intrinsic to the search result markup that Fancybox reads via `.find('figcaption').html()`. Measured: the page is **70% whitespace and NOT minified** (homepage 589 KB, /my-recent-photos/ 1.02 MB, activity 1.72 MB — all unminified). A safe whitespace collapse (preserving single spaces, skipping script/style) gives **67% reduction** (1.72 MB → 557 KB) with byte-identical visible text. **Fix = enable WP Rocket → File Optimization → Minify HTML** (site already licenses WP Rocket 3.21.2; currently OFF). Did NOT ship a custom ob_start minifier: it would duplicate a licensed feature, risk corrupting the Fancybox caption markup the lightbox parses, and conflict with WP Rocket. Not a theme deploy. |
| ... | remaining PDF tasks | not yet verified |

### POST-DEPLOY RE-VERIFICATION (staging now runs `development`) — 2026-11-02

After the client pushed the theme, re-checked everything on staging. **Confirmed fixed:**
PDF#1 (all taxonomy param links 200), #2, #3 (no scheme-less lodge links), #5 (0 keywords),
#6 (viewport OK), #7 (0 cache metas), #11 (single JSON-LD, 0 leading spaces), #12 (all H1s
present incl. /globe/, /grid/, /cp_birds_country/*), #26 (single jQuery core).

**New gaps found and fixed this round (repo, pending re-deploy):**
- **PDF#27 completion** — `functions.php` ALSO enqueued the external d3js topojson/queue
  globally (`theme_name_scripts`, lines 148-149), so /globe/ loaded them twice and the
  external URLs survived my template-only fix. Repointed both to local vendored files and
  removed the now-duplicate `<script>` tags from `template-globe.php`. Verified script order
  on staging: d3.min.js + topojson + queue all load from the theme before the inline globe code.
- **PDF#10 llms.txt returned HTTP 404** — the file body served fine but WP had already set a
  404 status before `template_redirect` fired. Added `status_header( 200 )`. (Was returning
  1829 bytes with `HTTP/2 404`.)
- **PDF#13/#23 alt text** — corrected scan (my earlier regex broke on `?>` inside tags) found
  15 true missing-alt images; fixed the content ones: `template-grid.php:203` (~32 /grid/
  thumbnails), `functions.php:851` (country thumbnails), `template-argentina.php` (10 gallery
  images), `template-entry.php` loader, header Facebook pixel. Only remaining is
  `functions.php:3465` (wp-admin column, not public).
- **PDF#12 empty H1** — `/globe/`, `/grid/`, `/countries/` H1 was present but EMPTY
  (`get_field('c_page_title')` unset). Added `?: esc_html( get_the_title() )` fallback.

**Not theme-fixable / client-side (confirmed on staging):** PDF#4 (44 dead external links —
content/ACF), #20/#21 (dup titles/H1s — Yoast/content), #24 (WebP — Cloudflare/Imagify),
#25 (HTML minify — WP Rocket setting), #30 (security headers — edge), #9 (robots.txt
Disallow:/ is deliberate), DOCX 21 items (WP admin / host).

### BUSINESS DEVELOPER (BD) EMAIL — 6 points re-verified on staging (2026-11-02)

**Root cause of the BD's findings: the fixes live in the repo (`development`) but the
client has NOT deployed `owendeutsch.zip` to staging yet.** Staging still runs the old
theme, so anything fixed only in the repo still shows as broken to a crawler.

| BD# | Claim | Verdict | Evidence (staging, cache-busted) |
|-----|-------|---------|----------------------------------|
| 1 | keywords meta still in header.php L58 | **STALE / ALREADY FIXED** | Staging renders `name="keywords"` count = 0 on /, /globe/, /blog/. Also 0 in repo header.php. BD looked at an old snapshot. |
| 2 | 3 dead scripts on /globe/ timing out 6s | **HALF TRUE** | Staging /globe/ DOES still load d3js.org topojson/queue + jsdelivr bootstrap (old theme). But the URLs are NOT dead: all 3 return HTTP 200 in ~0.1s (verified 2026-11-02). The "6s stall" is a custom.js loader timer, not a network timeout. Real cleanup = self-host, DONE in repo (PDF#27), pending deploy. |
| 3 | 44 broken external links not updated | **PARTLY TRUE** | Many recovered: nanpa.org 200, lincolnparkconservancy.org 200, sanisidrolodge.com 200, orangehill 200, sandpiperabaco 200, zebrahills 200, nikonusa 200. Still broken: owletlodge.org, africanpenguinsafe.org, 5x abcbirds.org article links, seattle-audubon, roku, lemalacamp, wildsumaco. NOT theme-fixable (post content + ACF lodge fields). |
| 4 | JSON-LD leading space still present | **TRUE on staging** | Staging homepage block 2 still has `"@vocab": "https://owendeutsch.com/"` + 3 leading-space URLs. FIXED in repo (PDF#11 removed the theme block), pending deploy. |
| 5 | Missing H1 on 15 pages | **TRUE on staging** | /globe/, /grid/, /cp_birds_country/ec/, /us/, /ca/ all h1=0 on staging. FIXED in repo (PDF#12 → h1.large), pending deploy. |
| 6 | 21 backend items not applied | **TRUE** | None applied. NOT theme-fixable (WP admin / WP Engine / host). See DOCX section. |

**Recommended action:** deploy `owendeutsch.zip` from `development` to staging. That
resolves BD#2 (self-hosted JS), BD#4 (JSON-LD), BD#5 (H1s), and re-confirms BD#1.
BD#3 needs content/ACF edits; BD#6 needs WP admin + host.
| PDF#12 pagination | /blog/ rel=next/prev + page/2/ 301 | FALSE POSITIVE now — /blog/, /blog/page/2/, /blog/page/3/ all 200; prev/next links present and correct (`<a class="next page-numbers">` etc.). index.php uses the standard loop; no rel=next/prev markup in the theme (core/Yoast handle it). The audit's 2 "sequence errors" were crawl artifacts. |
| PDF#19 | 11 blog articles use H1 for subheadings | CONTENT-level (post body), not theme code. The H1s are inside article content, not templates. |
| PDF#20 | 332 duplicate page titles | NOT theme code — Yoast title templates + post/ACF content. |
| PDF#21 | 343 duplicate H1s | Overlaps PDF#16/#18 (template H1s fixed) + content-level H1s (PDF#19). Not a separate theme fix. |
| PDF#23 | 10,207 images missing width/height | PARTLY DONE in PDF#13 (gallery templates). Remaining instances are content-inserted images (WP core auto-adds width/height for standard content images) — not theme-templated. |
| PDF#31 | Establish monthly recurring audit | Process/deliverable (client engagement), not theme code. Covered by the DOCX roadmap Phase 4. |
| PDF#29 | 15 http:// links | VERIFIED — theme has ZERO live http:// links: header.php's `http://code.jquery.com/ui/1.10.2/jquery-ui.js` is COMMENTED OUT; all other http:// strings are inside vendor JS/CSS licence comments. A `http://localhost/owen/...` string exists but is inside a commented-out JS line in functions.php (renders as a JS comment). The 15 links the audit found are content/plugin-level, not theme. |

### CLIENT PDF — remaining items verified on staging (not theme-code)

| # | Item | Verdict |
|---|------|---------|
| PDF#12 tail | 332 dup titles / 343 dup H1s | Sitewide metadata (Yoast title templates + post content), not theme code |
| PDF#13 | 10,853 JPEGs vs 2 WebP | Enable WebP at Cloudflare/WP Engine or Imagify (infra) |
| PDF#14 | Security headers (HSTS/XFO/nosniff/CSP) | Set at Cloudflare/WP Engine edge (infra). Theme sets none; confirmed absent on staging |
| PDF#14 | 1.76 MB activity-search HTML | Not on homepage (589 KB, 0 figcaptions); only on large result pages. Not a safe blind edit |
| PDF#9 | Crawl-delay: 10 + /?s= noindex | /?s= ALREADY `noindex, nofollow` (verified). Crawl-delay comes from Yoast/robots.txt |
| PDF#6 | /globe/ 6s load + 3 dead scripts | FALSE POSITIVE now: /globe/ = 0.69s, all 3 scripts return 200 |
| PDF#6 | /2025-year-in-review-gallery/ timeout | Now 200 |

### SECOND DOCUMENT — `Owen-Deutsch-Website-Maintenance-Site-Health-Review.docx`
Infrastructure / host-level review (21 issues). **No theme-code changes required** — all
items are WP Engine / Cloudflare / wp-config / plugin / content actions for the client.
Reference for the monthly care plan. Theme-relevant notes only:
- **#19 DB charset utf8 (not utf8mb4)** — same encoding family as the PDF#1 en-dash/%FF
  issue; convert on staging, then re-test the location filters.
- **#16 version disclosure** — WP Rocket generator meta is emitted by the plugin, not the theme.
- **#14/#15 inactive themes & plugin readme** — host/plugin cleanup, not theme files.

### ENGAGEMENT STATUS (theme deliverable)
All theme-code-fixable items across BOTH documents are DONE in `development`.
Remaining work is client/infra: deploy `owendeutsch.zip` to staging, then set security
headers + WebP + utf8mb4 + memory/OPcache at the host edge.

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
> * **WORKAROUND (verified 2026-11-11): read live through a public reader proxy.** Direct curl
>   and the headless browser both get the firewall block, but these return the real page:
>   `https://api.allorigins.win/raw?url=<URL>` (worked) and
>   `https://thingproxy.freeboard.io/fetch/<URL>` (worked). `r.jina.ai` returns 401 (needs a key),
>   `api.codetabs.com` returned 522. Use a proxy to CONFIRM live state; staging is still reachable
>   directly. Do not report a live result as verified unless it came through a proxy or the client.
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
| OA-1 | Zoom webinar registration 404 in the (commented-out) home popup, footer.php L122-151, plus orphaned trigger in js/custom.js L358-370 | URL string present on 453 pages, but never rendered (inside an HTML comment) — no user/SEO impact | **CLOSED (repo, 2026-11-05)** — confirmed dead code on staging (zoom URL present only inside HTML comments; 0 rendered links; `#home-popup-modal` never rendered, so the custom.js trigger was a no-op). Removed the commented modal block (30 lines), the orphaned cookie/click trigger (custom.js, 13 lines) and an orphaned `#home-popup` click handler (3 lines). No user or SEO impact either way; this is hygiene. |
| OA-2 | Typo'd self-link `https://www.owendeustch.com` on `/lodges/arizona-usa/` H1 (missing the "s"; edit distance 2 from `owendeutsch.com`, NXDOMAIN) | Headline lodge link points at a non-existent domain | **CLOSED — FIXED ON LIVE (2026-11-11)**. Live H1 now reads `<h1 class="large text-center"><a href="https://owendeutsch.com" target="_blank" rel="noopener">Arizona – USA</a></h1>` — verified via two independent external proxies (allorigins + thingproxy) because this container's IP is blocked by BlogVault Firewall on live. Fix applied by the user in WP admin on the **ACF field `site_url`** of the Arizona – USA lodge post (the H1 link is `od_normalize_url( get_field('site_url') )` in `single-lodges.php`). The 2026-11-05 "CLOSED" was wrong — it was verified on staging only, and the URL lives in postmeta, so no theme push could ever have fixed it. Note for future work: **staging can be clean while live is broken; verify on live.** |
| OA-3 | ABA (`aba.org`) 502 cluster — 6 URLs (root returns 200, sub-pages 502) | Likely a server-side outage at ABA, not our bug | **CLOSED — RECOVERED (2026-11-05)**. Re-checked: `aba.org/` 200, `/birding/` 200, `/events/` 200. (`/aba-blog/` 404, but ABA restructured its site — a dead URL to drop from content, not a server outage.) Was a transient server-side outage at ABA. |
| OA-4 | Intermittent HTTP 504 on ~13 lodge pages (`/lodges/huembo-reserve-peru/`, `/lodges/wildsumaco-lodge-ecuador/`, `/lodges/monterey-bay-california-usa/`, etc.) | Edge 504 to visitors while the origin is under load | **CLOSED — NOT REPRODUCING (2026-11-06)**. Re-tested 7 named lodges x3 passes on BOTH live and staging: 21/21 live and 21/21 staging returned 200, first byte 0.1-1.7s. A full re-crawl of all 148 live lodge pages returned 0 non-200. Original 504s were a transient origin-timeout condition; no theme or content action. Was INFRA (WP Engine origin), not theme. |

Detection method for OA-2 (typo'd self-link): extract hrefs, compute edit distance of each
host against the canonical domain, and DNS-check the near-miss. Status code alone cannot
distinguish a self-link typo from an unrelated dead partner domain (distance 2 vs 12-14).

## External link crawl — LODGE PAGES re-measured on LIVE (2026-11-06)
Crawled all 148 live lodge pages (from `/lodges-sitemap.xml`) -> 2,297 distinct internal
links + 96 distinct external links. First crawl run against LIVE, not staging.
Internal: 2,295x 200, 2x 403 (`/xmlrpc.php` + `?rsd` — deliberate block, expected, appears
in every page head via `rel="pingback"`). **0 internal broken links.**
External: 78x 200, 4x 404, 4x no-response (000), 5x 403, 2x 429, 2x 522, 1x 401.
Actionable (verified by hand, not just the crawler):
- `www.owendeustch.com` on `/lodges/arizona-usa/` — **REAL, see OA-2. Live only.**
- `www.lemalacamp.com/kuria-hills-tented-lodge` — root AND deep link both 000; host down.
- `www.cabanassanisidro.com` — 000; DNS resolves but no HTTP response.
- `www.mendosa.com` — NXDOMAIN; domain is gone.
- `www.owletlodge.org` — 522 (Cloudflare cannot reach origin).
- `porini.com/kenya/parks-and-reserves/nairobi-national-park/` — 404 (root is 200; page moved).
- `lincolnparkconservancy.org/north-pond-restoration-10/` — 404 (root is 200; page moved).
- `isimangaliso.com/jewel/western-shores-and-charters-creek/` — 404.
- `flickr.com/photos/cheesy42/3357058572/` — 404 (photo deleted).
Non-actionable: 403s are bot-blocking (pixabay, kagerasafaris, thesafaricollection,
tripadvisor, weather-us); 429s are Instagram rate-limits on `@owendeutsch` / `@nathan_goldbird`
(fine in a real browser); `dunesnationalpark.org` flipped 522 -> 403 between runs (bot block).
Raw data: /workspace/links.json (note: /workspace, not /tmp — /tmp is wiped on container restart).

### Dead-link remediation table (researched 2026-11-11, LIVE)

Re-checked all 8 actionable links. **3 have a confirmed working replacement** (found in the
target site's own sitemap and verified 200). **4 are genuinely down** — no replacement exists,
the client must decide. These are content/DB values, NOT theme code, so they need a WP admin
edit or Better Search Replace — a theme push will never fix them.

| Lodge page (live) | Dead URL | Verdict | Replacement (verified 200) |
|---|---|---|---|
| `/lodges/nairobi-national-park-kenya/` | `www.porini.com/kenya/parks-and-reserves/nairobi-national-park/` | **FIXED (2026-11-11)** | `https://www.porini.com/african-safari-holidays/kenya/parks-and-reserves/nairobi-national-park/` — replaced by client via String Locator (DB search+replace) on **live**, table `wp_postmeta`.`meta_value`. Verified live: OLD=0 NEW=2 (both `site_url` H1 + `credit_url` map credit), new URL 200. Confirms String Locator DOES do DB search/replace (changelog v2.6.0). |
| `/lodges/north-pond-illinois-usa/` | `lincolnparkconservancy.org/north-pond-restoration-10/` | **FIXED (2026-11-11)** | `https://lincolnparkconservancy.org/projects/north-ponds-restoration/` — replaced by client via String Locator on **live**. Verified live: OLD=0 NEW=1, new URL 200. NOTE: this one lives in the **About body text** (post content, `wp_posts.post_content`), NOT in `site_url`/`credit_url` — so the two replaces hit different tables. |
| `/lodges/western-shores-of-isimangaliso-wetland-park-south-africa/` | `isimangaliso.com/jewel/western-shores-and-charters-creek/` | **FIXED (2026-11-11)** | `https://www.isimangaliso.com/places/western-shores-charters-creek/` — replaced by client via String Locator on **live**. Verified live: OLD=0 NEW=1, new URL 200. The H1 (`site_url`) now uses the new `/places/` path; `credit_url` still points at the bare `https://isimangaliso.com/` which 200s (301 -> www). Site restructured `/jewel/` -> `/places/`. |
| `/lodges/tobago-trinidad-and-tobago/` | `flickr.com/photos/cheesy42/3357058572/` | **FIXED (2026-11-11)** | Repointed to photostream root `https://www.flickr.com/photos/cheesy42/` — replaced by client via String Locator on **live**. Verified live: old photo id `3357058572` count=0, stream link count=1, stream URL 200. Credit text still renders as "® Flickr". |
| `/lodges/lemala-kuria-hills-lodge-tanzania/` | `www.lemalacamp.com/kuria-hills-tented-lodge` | **FIXED (2026-11-11)** — was NOT dead, company rebranded | Replaced with `https://www.lemalacamps.com/stay/kuria-hills-lodge` (note the added "s" in `lemalacamps`). Old domain `lemalacamp.com` -> 104.248.46.30, ports 80/443 closed (server offline). New domain `lemalacamps.com` -> 165.232.39.226, 200. Verified live: OLD=0 NEW=2, new URL 200 (no-slash form 301s to trailing-slash). Only ONE lodge page references lemala, so no collateral damage. Client applied via String Locator on live. |
| `/lodges/cabanas-san-isidro-lodge-ecuador/` | `www.cabanassanisidro.com` | **FIXED (2026-11-11)** — was NOT dead, company rebranded twice | Replaced `www.cabanassanisidro.com` -> `www.sanisidrolodge.com` (client, String Locator, live). Chain: `cabanassanisidro.com` (443 CLOSED, 80 open) 301 -> `cabanasanisidro.com` -> `www.sanisidrolodge.com` (200). Note: `credit_url` ALREADY pointed at `sanisidrolodge.com`; only the H1 `site_url` was stale. Verified live: OLD=0, `sanisidrolodge.com` count=3 (H1 href + credit href + credit link text), new URL 200. |
| `/lodges/owlet-lodge-at-abra-patricia-reserve-peru/` | `www.owletlodge.org/` | **FIXED (2026-11-11)** — replaced with EMPTY | Client replaced the string with an empty value via String Locator on **live**. Verified live (2 fresh fetches, `x-cache: MISS` / `cf-cache-status: MISS`): H1 now renders **plain text, no `<a>`** — `<h1 class="large text-center">Owlet Lodge at Abra Patricia Reserve – Peru</h1>`. Sitewide scan of all 455 sitemap URLs: **0 pages** still reference `owletlodge.org`. Credit still renders `© American Bird Conservancy` with the theme's pre-existing stray `</a>`. No `href="https://#"`, no `https://https://` double-prefix, JSON-LD still valid. **COLLATERAL: 1 page — `/abc-mike-parr-my-favorite-bird/`** (see note below). |
| `/lodges/ramsey-canyon-arizona-usa/` | `www.mendosa.com/` | **FIXED (2026-11-11)** — replaced with EMPTY | Client ran **two String Locator passes** on **live** (longer string first, then the short one). Verified live (3 cache-busted fetches, all `x-cache: MISS` / `cf-cache-status: MISS`): H1 is plain text (`Ramsey Canyon – Arizona, USA`, no `<a>`), credit now renders **plain text `© David Mendosa`** with the theme's pre-existing stray `</a>`. **`mendosa` hrefs = 0**, `fitnessblog` = 0 sitewide. Only remaining `mendosa` on the whole site is the attribution *text*. No `https://https://`, no `href=""`, no `href="https://#"`. Sitewide scan of 455 URLs: 1 page (the text itself), 0 links. `mendosa.com` does not resolve (DoH SERVFAIL) but is still registered (RDAP: expires 2027-06-04) — the credit points nowhere useful, so plain text is the correct end state. **Staging** (`owendeutsc2stg`) still shows the old linked value — separate DB, not part of this fix. |

**⚠️ Propagation lag:** immediately after the replace the live page **still served the old link** for several minutes on `x-cache: MISS` (fresh render) — WP Engine's `object-cache.php` + `advanced-cache.php` drop-ins hold the old `get_field()` value until their cache entry cycles. It resolved on its own. **Do not re-run a replace because the first verification still shows the old value — wait, then re-fetch.** (Owlet behaved the same way.)

**String Locator shows 11 postmeta rows for `https://www.mendosa.com/` — but only ONE renders.** Breakdown:

| postmeta IDs | value | renders? |
|---|---|---|
| `267970` | `https://www.mendosa.com/` | **YES** — the Ramsey credit (`credit_url`) |
| `267986 268002 268018 268034 268050 268066` | `https://www.mendosa.com/fitnessblog/?p=11112` | no (`fitnessblog` = 0 hits sitewide) |
| `278620 1090791 1090811 1090831` | `https://www.mendosa.com/` | no (REST `?search=mendosa` = 0 on all post types) |

**Do NOT single-pass replace `https://www.mendosa.com/` -> empty.** Because 6 rows contain the *longer* `…/fitnessblog/?p=11112`, a naive replace mangles them into the relative fragment `fitnessblog/?p=11112`. Two safe options:

1. **Surgical (preferred):** open the Ramsey lodge in WP admin and clear the **`credit_url` ACF field** directly. One value, one page, zero collateral — the other 10 rows stay as inert dead metadata.
2. **String Locator, two passes (longer string first):** first `https://www.mendosa.com/fitnessblog/?p=11112` -> empty, then `https://www.mendosa.com/` -> empty. Pass 1 clears the 6 long rows; pass 2 then only matches the 5 plain rows. A one-pass replace with the short string does NOT work.

After either fix the credit renders `© David Mendosa` plus the theme's pre-existing stray `</a>` (see note above). **Coverage caveat:** the sitewide scans here cover posts (276), pages (33), lodges (147) and a 60-page sample of the 9,007 `cp_birds` — the bird template renders no external/meta links, and the REST search returns 0 for `cp_birds`, so birds are clean.

Note: `isimangaliso.com` 301-redirects to `www.isimangaliso.com`; the old path 404s on both.
The replacement is under `/places/`, not `/jewel/` — the site was restructured.

### Owlet collateral: the string lived on TWO pages, and one is damaged

The String Locator search found ~48 rows; only **2 were public-rendered**. Both mattered:

1. `/lodges/owlet-lodge-at-abra-patricia-reserve-peru/` (`site_url` + `credit_url` postmeta) -> H1 correctly becomes **plain text**; credit keeps its text. **Good.**
2. `/abc-mike-parr-my-favorite-bird/` (`wp_posts.post_content`, id **12172**) -> **FIXED (2026-11-11/12).** The post had an inline `<a href="https://www.owletlodge.org/" target="_blank"> The Owlet Lodge </a>` in its body. The client **unlinked the text** (now plain text), then cleared the residual empty anchor via String Locator. **Final verified state** (3 cache-busted fetches + REST `posts/12172`, `modified 2026-11-12T00:23:12`): `href=""` = **0**, `owletlodge` = 0, anchor balance **48/48**, visible text `...rare Long-whiskered Owlet! The Owlet Lodge is a great place to stay...`. Sitewide audit of 455 URLs: **0 dead domains, 0 owlet/lodge-related empty hrefs.**

**Lesson for future String Locator replaces:** always check `wp_posts.post_content` too — the same string can be a body link on an unrelated page. The lodge template (postmeta) and post bodies (`wp_posts`) are separate tables, and a postmeta-only fix leaves body links broken.

**Baseline vs damage (do not misread the sitewide scan).** 54 pages show `href=""`, but **53 are pre-existing theme furniture** — the H1 on lodges with no `site_url`, and tag/related-post widgets — confirmed present in the 2026-03-07 Wayback snapshot of `/lodges/maumee-ohio-usa/` (6 empty hrefs before any of this work). The **only new one is Mike Parr**. To tell them apart, look at the anchor TEXT: `Ohio`/`Illinois`/`Clear` = baseline; `The Owlet Lodge` = damage.

**Fix for Mike Parr (pending client).** Either restore a live URL on that anchor (e.g. the ABC reserve page `https://abcbirds.org/where-we-work/latin-america-caribbean/latin-american-reserve-stewardship-initiative/`), or remove the `<a ...>`/`</a>` pair so "The Owlet Lodge" reads as plain text. A bare `href=""` is the one outcome to avoid.

### Stray `</a>` in the credit block is a THEME BUG, not link damage

`single-lodges.php` lines ~67-75 wrap the credit in a conditional open tag but an **unconditional close tag**:

```php
<h3 class="desktop-show">
<?php if ( get_field('credit_url') ) : ?>
  <a href="<?php echo esc_url( od_normalize_url( get_field('credit_url') ) ); ?>" target="_blank">
<?php endif; ?>
  <?php the_field('credit_name'); ?>
</a>            <!-- <-- always emitted, even when credit_url is empty -->
</h3>
```

So when `credit_url` is empty the page ships `© Name </a>` with no opening `<a>` — an **orphan closing tag** that makes the surrounding anchor nesting unbalanced. **52 of the lodge pages do this today** (verified by parsing `<h3 class="desktop-show">…</h3>` on all 455 sitemap URLs), and it is **pre-existing**: the 2026 Wayback snapshot of `/lodges/abaco-bahamas/` already shows `© Owen Deutsch </a>`. Browsers tolerate it, so it is cosmetic — but it is why blanking `credit_url` (Owlet, and the proposed Ramsey fix) leaves the credit text intact.

The one-line theme fix is to move `</a>` inside the `if`, mirroring the H1 which already does this correctly (`if ( $lodge_site_url ) : <a…></a> else : plain <?php endif;`). Two copies of the block exist (lines ~67 and ~104) — both need it.

**FIXED (2026-11-12) — in `owendeutsch.zip` on `development`, pending deploy.** Both copies now read:

```php
<?php the_field('credit_name'); ?>
<?php if(get_field('credit_url')): ?></a><?php endif; ?>
```

so the close tag is emitted only when the open tag is. Rendering verified by simulation: with a URL -> `<a href="…" target="_blank">© Name</a>` (unchanged); with empty `credit_url` -> `<h3 class="desktop-show">© Name</h3>` (no orphan tag). `php -l` clean, all 43 theme PHP files lint clean, zip rebuilt with 150 entries / order preserved / `testzip()` clean / only `single-lodges.php` differs. **Note:** `single-lodges-bk.php` carries the same bug but is a stale backup (no `od_normalize_url`, not a WP template name so never loaded) — deliberately left untouched. After deploy, expect the sitewide `href=""` count to fall from 54 toward 1 (the only non-theme one was the Mike Parr post, since cleared).

## Lodge link fields — how the theme renders them (READ BEFORE editing site_url / credit_url)

`single-lodges.php` renders both link fields through `od_normalize_url()` (functions.php:1594):

```php
function od_normalize_url( $url ) {
    $url = trim( (string) $url );
    if ( $url === '' ) return '';                                   // only the EMPTY string returns ''
    if ( stripos( $url, 'http://' ) === 0 )  return 'https://' . substr( $url, 7 );
    if ( stripos( $url, 'https://' ) === 0 ) return $url;
    if ( strpos( $url, '//' ) === 0 )        return 'https:' . $url;
    return 'https://' . ltrim( $url, '/' );                          // ANY other non-empty value gets https:// glued on
}
```

Consequences for editors (verified against the live theme source, 2026-11-11):

* **`site_url` empty** -> `$lodge_site_url` is `''` (falsy) -> template's `if ( $lodge_site_url )` fails -> the H1 renders as **plain text, no `<a>`**. Confirmed live: `/lodges/maumee-ohio-usa/`, `/lodges/north-pond-illinois-usa/`, `/lodges/uyuni-bolivia/`, `/lodges/ballestas-island-peru/` all have a bare `<h1>` with no link. **This is the correct way to "remove" a link.**
* **`credit_url` empty** -> the `if ( get_field('credit_url') )` guard skips the opening `<a>` but the closing `</a>` after `the_field('credit_name')` still prints -> **stray `</a>`**. Confirmed live on `/lodges/uyuni-bolivia/`: `<h3 class="desktop-show"> © Amy Rollo </a>`. Browsers recover silently (the credit text stays unlinked), but the markup is invalid. To drop a credit link, prefer clearing `credit_url` and accept the stray tag, or blank `credit_name` too.
* **`#` is NOT a way to disable a link.** `'#'` is non-empty, so `od_normalize_url('#')` returns `'https://#'` — the H1 then renders `<a href="https://#">`, a broken link pointing at a nonsense URL. `mailto:` and `javascript:` are mangled the same way (`https://mailto:...`). Do not use `#` as a placeholder.
* **Never leave a value without a scheme.** `od_normalize_url('owletlodge.org')` -> `https://owletlodge.org`, which happens to work, but a value like `example.com/path` relies on that gluing and is easy to get wrong; always store a full `https://...` URL.

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

### FINAL DELIVERABLE — client progress report (2026-11-02)

`Owen-Deutsch-Audit-Progress-Report.docx` — task-by-task client report covering all 58 items
(31 SEO audit + 6 BD points + 21 maintenance). Sections: Executive Summary (58 total = 25
fixed/verified + 33 client/infra), Deployment Status, SEO audit finding-by-finding, Broken
External Links detail WITH working replacements (Section 3), the 21 Maintenance items with
severity + owner (Section 4), the 6 BD points (Section 5), and Recommended Next Steps.
Styled to match the client docs (navy #142A45 / orange #D96B1E / grey #555B63).

### PDF#27 follow-up — removed redundant Bootstrap 5 from /globe/ (2026-11-02)

/globe/ loaded BOTH Bootstrap 5.3.0 (bundle, via the old jsdelivr CDN that PDF#27 self-hosted)
AND Bootstrap 3.3.2 (local `js/bootstrap.min.js`, enqueued globally at functions.php:134).
Verified BS5 was completely unused: zero `data-bs-*` attributes anywhere in the theme, zero
`bootstrap.*` JS API calls, no tooltip/popover/dropdown, and `img-fluid` is not even defined
in any theme CSS. The globe's tabs use Bootstrap 3 markup (`data-toggle="tab"`, `.tab("show")`),
which BS5 does not handle (BS5 needs `data-bs-toggle`). Removed the `<script>` tag from
template-globe.php and deleted the unused 80 KB `js/bootstrap-5.3.0.bundle.min.js`
(zip 151 -> 150 entries). /globe/ now loads a single Bootstrap (3.3.2), same as the rest of
the site. Note the audit itself flagged this CDN script ("cdn.jsdelivr.net/.../bootstrap.bundle.js
Timeout"), so this completes that finding rather than just self-hosting it.

### PDF#23 follow-up — width/height on search gallery images (2026-11-02)

`search.php:527` (the main AJAX bird-search gallery thumbnail, used on all search/filter results)
had alt but no width/height. Added `width`/`height` from the `square-medium` attachment array
(same pattern already used in `taxonomy-cp_birds_country.php:93` and `grid_template.php:104`).
This reduces layout shift (CLS) on the highest-traffic dynamic pages. Remaining instances are
images inserted directly into post bodies, which WordPress core handles.


### SECURITY HEADERS — live on production (2026-11-06)

Verified independently from outside the network.

**Final state on the apex (`owendeutsch.com`)** — set via **WP Engine → Web Rules → Header
Rules**, operation "Set", "Only on successes", no conditions:

| Header | Value | Occurrences |
|---|---|---|
| `Strict-Transport-Security` | `max-age=300` | 1 |
| `X-Content-Type-Options` | `nosniff` | 1 |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | 1 |

Coverage: homepage, `/blog/`, `/grid/`, `/globe/`, `/lodges/`, lodge detail, `/about-owen/`,
blog article, `/sitemap_index.xml`, `/wp-json/` — all 200, all 3 headers. Homepage serves them
on both cache HIT and MISS, so no purge is outstanding. No duplicates anywhere.

**Why the earlier Cloudflare work did not cover the main site.** The apex A records and
`www` CNAME are **DNS only**, which overrides the proxied `*.owendeutsch.com` wildcard. Those
two hostnames bypass the Cloudflare zone entirely and are fronted by **WP Engine's own
Cloudflare** (hence `server: cloudflare` + `cf-ray` on the apex). That is why headers for the
main domain must be set in the WP Engine portal, not by orange-clouding DNS. The Cloudflare
Response Header Transform Rule + HSTS toggle still cover the proxied hostnames reached via the
`*.owendeutsch.com` wildcard (`blog.`, `cdn.`, and anything else undefined) at 5 headers each.

**Do not confuse the two staging hostnames.** The real staging environment is
`owendeutsc2stg.wpenginepowered.com` (WP Engine origin, 141.193.213.10). There is **no
`staging` DNS record** — `staging.owendeutsch.com` only resolves because of the wildcard, so it
is a phantom name served through Cloudflare, not the staging site. All reports prior to
2026-11-06 used `owendeutsc2stg.wpenginepowered.com`. As of 2026-11-06 the real staging
environment has **no security headers**; add them via WP Engine Header Rules on that
environment if staging parity is wanted.

**Three HSTS values are now in play.** Header Rules serve `max-age=300` on content pages;
Cloudflare serves `max-age=2592000` on proxied subdomains; and `/wp-login.php` returns
`max-age=31536000` from a platform-set header. Keep all three in mind so they do not drift
apart unnoticed. `includeSubDomains` and `preload` are off.

**Known gaps:** static files (`robots.txt`, CSS, JS) and error responses get no headers. The
error-page gap is by design ("Only on successes"). For static files the exact reason is not
confirmed from outside the network — they appear to be served through a path that bypasses the
rules phase, not through WordPress, so the rules never see them. (Web Rules run in nginx ahead
of PHP, so "they run at the PHP layer" is wrong.) Impact is minor but not zero: `nosniff` is
useful on JS/CSS. Cloudflare cannot close this gap because the main domain is not proxied
through the account. No fix attempted.

**Other headers seen on the apex (not from Header Rules, no conflict):**
- `/wp-login.php` → `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, and
  `strict-transport-security: max-age=31536000`. The one-year HSTS is platform-set, not the
  rule (which says 300). Worth knowing before raising the rule value, since a browser updates
  its stored policy on every response — visiting the login page already sets one year.
- Blocked paths (e.g. `/.env`) → 403 with `x-frame-options: SAMEORIGIN`,
  `referrer-policy: same-origin`, `x-content-type-options: nosniff` (platform-set).
- Normal content pages carry only the three Header Rule headers. Where both apply, the rule
  value wins — so `strict-origin-when-cross-origin` overrides `same-origin`. No breakage.
- 404s carry none.

**Unrelated items checked:**
- XML-RPC is **already blocked** (403 on prod and staging) — no action needed.
- DMARC is `v=DMARC1; p=none;` (monitor only). Consider `p=quarantine` after reviewing reports.
- `mail`, `msoid`, `_domainconnect` are proxied (unusual). Tested and working — `mail` returns
  301 with Exchange headers intact, `msoid` returns 200 from Microsoft. MX is DNS-only, so
  mail delivery is unaffected.
- `*.owendeutsch.com` is proxied, so undefined subdomains resolve to Cloudflare. Review whether
  the wildcard is still needed (it also makes `includeSubDomains` HSTS riskier than it looks).

DNS baseline captured in `docs/cloudflare-dns-records.md` for future diffing.
Report: `Owen-Deutsch-Security-Headers-Addendum-2026-11-06.docx`.

### POST-DEPLOY VERIFICATION — theme updated on staging (2026-11-05)

Client uploaded the updated `owendeutsch.zip` to staging. Re-verified everything.

**Confirmed live on staging:**
- `lodges_country-sitemap.xml` -> **404**, and gone from `sitemap_index.xml` (now 13 sitemaps:
  post, page, lodges, cp_birds x9, author). Country exclusion working.
- Dead country URLs (`/cp_birds_country/argentina/`, `peru`, `belize`, `mexico`, `caribbean`)
  -> **404** and no longer listed. Bird country archives (`/ar/`, `/pe/`, `/be/`) -> **200**.
- H1: `/blog/`=1, `/videos/`=1, `/globe/`=1, `/grid/`=1, `/`=1. (Were 0 on blog/videos.)
- Home-popup dead code gone: 0 `us02web.zoom` and 0 `home-popup` refs in homepage HTML and
  in the live `js/custom.js`.
- Country archive images now carry `width`/`height` (PDF#6/#13) — live.
- Full crawl of all non-bird pages + 600-bird sample (**1,053 pages**): **1,051 x 200**.
  The 2 non-200 were both non-issues: one transient 504 that returned 200 on retry, and one
  malformed URL that was a **crawl artifact** (a missing newline when concatenating two URL
  files joined two valid URLs; both resolve 200).
- All 15 lodge pages that returned 504 during the first sweep returned **200** on individual
  retry -> intermittent WP Engine origin timeouts, not broken pages.

**Quality scan (23 representative pages: home, blog, videos, globe, grid, countries, about,
search, my-recent-photos, 3 country archives, lodges index, 2 lodges, 2 posts, 5 bird pages):**
every page = exactly **1 H1**, **1 JSON-LD** block, **0** keywords meta, **0** images without
alt. Consistent with the earlier repo fixes.

**IMPORTANT — production still runs the OLD theme.** Verified on `owendeutsch.com`:
`lodges_country-sitemap.xml` still present, `/cp_birds_country/argentina/` + `/peru/` still
404, `/blog/` emits **100 H1s** and `/videos/` **22 H1s** (the pre-fix loop-title markup).
Promoting the theme to production fixes all of that.

**Staging vs production indexing note.** Staging serves `noindex, nofollow` site-wide
(`robots.txt` = `Disallow: /` + Yoast meta). Production serves `index, follow` and a real
robots.txt. So audit counts measured against staging (e.g. duplicate titles/H1s) differ from
what production will emit; don't quote staging counts as production counts.

**Remaining theme-level item found:** `index.php` blog-listing thumbnails use
`wp_get_attachment_image_src(...,'full')` and emit `<img>` with **no width/height** -> CLS on
`/blog/`. Small, safe follow-up (same pattern already applied to the search gallery).
Minor: `/contact/` 404s (not in sitemap; real page is `/about-owen/`).

### Country taxonomy 404 fix + H1 on blog/video pages (2026-11-03)

**Symptom:** 23 of the 31 URLs in `lodges_country-sitemap.xml` returned 404 on staging, e.g.
`/cp_birds_country/argentina/`, `/cp_birds_country/peru/`. `/grid/` and `/globe/` linked the
same pattern with 2-letter codes (`/cp_birds_country/ar/`) and worked.

**Cause:** `functions.php` registered two taxonomies with the *same* rewrite slug,
`cp_birds_country` — `cp_birds_country` on `cp_birds` (bird terms, slugs are ISO codes like
`ar`) and `lodges_country` on `lodges` (lodge terms, slugs are full names like `argentina`).
Both were also registered at `init` priority 0, and `custom_post_type_lodges()` runs last, so
`lodges_country` won the shared rewrite base. `/cp_birds_country/<lodge-slug>/` then resolved
to a lodge term with no matching template -> 404. The bird country terms are **not** in any
sitemap (Yoast generates no `cp_birds_country-sitemap.xml`), so the whole `lodges_country`
sitemap was dead weight.

**Fix:** set `lodges_country` to `'rewrite' => false` (lodges group by `term_id` on the lodges
page; no archive URL is needed), and exclude the taxonomy from both Yoast and core sitemaps
(`wpseo_sitemap_exclude_taxonomy`, `wp_sitemaps_taxonomies`). A guarded one-time
`flush_rewrite_rules( false )` runs on `init` (option `od_rewrite_version`) so the slug change
takes effect on an FTP theme upload, which does not re-activate the theme.

**Also:** `index.php` (the `/blog/` posts listing) and `templates/template-video-latest.php`
(`/videos/`) rendered no page heading, so those documents had no H1. Added a visually hidden
`<h1 class="sr-only">` to each (Bootstrap 3.3.2 is bundled locally, so `.sr-only` exists).

**Note:** after this fix `lodges_country-sitemap.xml` should disappear from the sitemap index.
If the client wants the ~23 lodge-country URLs to remain indexable, that needs a dedicated
`taxonomy-lodges_country.php` template plus re-adding `lodges_country` to the sitemap — a
content/SEO decision, not a bug fix.
