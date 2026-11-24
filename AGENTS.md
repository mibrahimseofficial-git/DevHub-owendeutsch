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

These two are the client's own source documents and are the ONLY documents kept in the
repo. All progress/verification/addendum reports we generated were removed from
`development` on 2026-11-12 (client request) — see the note in the FINAL DELIVERABLE
section. If asked for a report again, generate it fresh (do not commit it unless asked).

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
- **Task 2 (birdingtop500 footer badge):** REAL, CLOSED (staging) — **REVERSED 2026-11-16
  at client request: badge RESTORED and made live (uncommented).**
  Original 2026-11-02 outcome: block removed from `footer.php` in the repo zip (commit
  `7e40cec`), matching staging. Rationale then: badge server was dead (HTTP 000, 20s
  timeout) and the block had been pre-commented in the client's original upload, so it
  never rendered for visitors.
  **2026-11-16 update:** birdingtop500.com came back online. Verified live:
  `https://www.birdingtop500.com/` = 200; the real badge image
  `https://www.birdingtop500.com/button.php?u=owendeutschphotography005` = **200,
  `image/png`, 24,199 bytes**. (Note the exact username `owendeutschphotography005` —
  the shorter `?u=owendeutsch` returns 404.) Client asked for the badge back, so the
  markup was restored to `footer.php` in the repo zip, **uncommented and functional**
  (adds `target="_blank" rel="noopener"`; the original had no rel and used `<br />`).
  It is no longer dead code. Placement: inside `<p class="site-by">`, above the
  copyright line. Deploy = theme zip to WP Engine, then purge WP Rocket -> WP Engine ->
  Cloudflare. **STILL PENDING: deploy to staging, client sign-off, then LIVE.**

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

### LIVE PLUGIN AUDIT AFTER CLEANUP — 2026-11-24 (client supplied wp-admin plugin list)

**24 plugins installed, 22 active, 2 inactive.** Client deleted 11. Verified live post-cleanup:
all pages 200, no broken assets (26 referenced, 0 broken; one transient 429 on loader.svg that
retried 200).

#### 🔴 MapSVG PRO DELETED — tables survived, but this is now a landmine
Client deleted **MapSVG Pro**. Free **MapSVG 9.0.13** remains but is **INACTIVE** and shows
"You cannot delete this plugin as other plugins require it / Required by: MapSVG Pro" — a stale
dependency record now pointing at a deleted plugin.

**The theme queries MapSVG's tables directly** (`wp_mapsvg_regions_486`, `wp_mapsvg_schema`) in
`functions.php` (L649, L660, L2320, L2345) and in two **`nopriv` AJAX handlers**:
- `wp_ajax_nopriv_get_country_bird` — used by `/globe/` to colour enabled countries
- `wp_ajax_nopriv_get_country_code` — used by the globe click-through

**Verified still working right now** (tables were NOT dropped by the plugin deletion):
- `get_country_bird` → 200, 3,307 bytes, returns AR/BS/BE/BZ… all `status:1`
- `get_country_code&cnt_id=4` → `{"status":true,"cnt_code":"AR"…}`
- `/globe/` 200 (77,912 B), `/countries/` 200 (78,472 B), `/cp_birds_country/{ar,be,br,ca,us}/` all 200

**So the site is fine today, but:**
1. **Never run "delete unused tables" / a MapSVG uninstall cleanup** — it would drop
   `wp_mapsvg_regions_486` and `wp_mapsvg_schema`, and the globe would silently lose all country
   highlighting and click-through.
2. **Never delete the inactive free MapSVG plugin** — its admin screens are the only UI for that
   table. Without it the data is editable only via SQL.
3. **The tables are now orphaned** — no plugin manages them. Add a **DB backup before any
   WP Engine "clean up" or hosting migration**.
4. If the client wants the dependency warning gone, deactivate is already done; do **not** click
   Delete on MapSVG free.

#### ⚠️ WP Rocket appears to be gone — contradicts earlier assumption
`wp-content/plugins/wp-rocket/` → readme **404**, no `wp-rocket` string in page HTML.
Homepage cache headers show only `x-cache: MISS`, `cf-cache-status: MISS`, `x-cacheable: SHORT` —
**no WP Rocket signature**. AGENTS.md previously assumed WP Rocket 3.21.2 was licensed and active.
**It is not in the current 24-plugin list.** Either it was removed or it was never on this install.
**Consequence:** the documented purge chain ("WP Rocket → WP Engine → Cloudflare") is wrong now —
purging must be **WP Engine cache → Cloudflare** only.

#### ✅ Autoptimize 3.1.17 is doing CSS optimisation
Homepage references 6+ `autoptimize/css/autoptimize_single_*.css` files. This supersedes part of
the earlier "enable WP Rocket minify" recommendation for **CSS**. HTML is still **unminified**
(208,868 B, 148 newlines) — Autoptimize's HTML minify is not on, or is not applying.

#### Page health after deletions (all 200)
`/` 208,868 · `/globe/` 77,912 · `/grid/` 65,236 · `/countries/` 78,472 · `/blog/` 154,200 ·
`/entry/` 61,443 · `/conservation/` 65,007 · `/birding-sites-and-lodges/` 214,228 ·
`/my-recent-photos/` 335,849 · `/llms.txt` 1,525 · `/?s=toucan` 200 (359,008)

#### ⚠️ Search URL regression — `/search/?s=toucan` → 404
`/search/` alone → 200, `/?s=toucan` → 200, but **`/search/?s=toucan` → 404 (45,743 B)**.
The theme's `od_llms_txt()` advertises `/search/` as a core resource. Someone appending a query to
the pretty URL gets a 404. Likely the `preg_replace` on `posts_where` in `cus_posts_where()` or
`rlv_manage_queries()` mangling the query. **Needs investigation** — real user-facing bug.

#### Plugin inventory — 24 installed
**Active & required (do not touch):** ACF Pro 6.8.12, Autoptimize 3.1.17, Bird Analytics 1.0.0,
Category Order & Taxonomy Terms Order 2.1, Entry Counter 1.0, Forminator 1.58.6, Hide Featured
Image 1.3.1, Imagify 2.3.5, Migrate Guru 6.72, Missed Schedule Post Publisher 2.4, Past Newsletter
Entries 1.0, Popup Maker 1.25.0, reCaptcha by BestWebSoft 2.1, Redirection 5.10.1, Regenerate
Thumbnails 3.1.6, Smash Balloon Instagram Feed 6.17.0, String Locator 2.6.8, BlogVault 6.85,
WP Activity Log 5.7.0, YITH Infinite Scrolling 2.14.0, Yoast Duplicate Post 4.7, Yoast SEO 28.9
**Inactive:** MapSVG 9.0.13 (**keep** — see above), Wordfence 9.1.1 (**activate for 2FA**)

**Deleted by client:** Akismet, Classic Editor, Export All URLs, MapSVG Pro, Reorder by Term,
ReOrder Posts within Categories, Slider and Carousel Plus Widget for Social Media, Tumblr Importer,
WP All Export, WP Content Copy Protection, WP Crontrol, WP Import Export Lite

#### ⚠️ Akismet was deleted
Akismet is now gone. The **Comment Cleaner** class in `functions.php` is the only spam control
left, and commenting is already disabled. Acceptable — but if commenting is ever re-enabled,
spam filtering must be reinstated first.

#### ⚠️ Wordfence: installed but INACTIVE on live
Live now has **Wordfence 9.1.1 installed, Deactivate/Delete shown = currently INACTIVE.**
(Earlier probe returned 404 on readme.txt and I wrongly reported "not installed" — the readme is
not served on this host; `.php` returns 403 for every path, so it is a useless signal.)
**2FA is therefore still OFF on live.** Activating Wordfence is step 1 of the 2FA rollout.

#### Staging is now far behind
Staging Forminator **still 1.53.1** (vulnerable). Staging still has the slider plugin active, the
comment feed open, the spam widget, and the static BirdingTop500 PNG.

### 2FA ROLLOUT — "DO NOT REMOVE ANY USER ACCOUNTS" — 2026-11-24

Client (Claire) constraint: **every existing WP user account must remain untouched.** Only 2FA,
cleanly, no lockouts. Nothing below deletes, edits, demotes, or re-passes any account.

**2FA plugin state (probed both envs):**
| Env | Wordfence | Any other 2FA plugin |
|---|---|---|
| **LIVE** | **404 — not installed** | none (probed 9 candidates) |
| **STAGING** | **present, v9.1.1** | none |

**LIVE HAS ZERO 2FA TODAY.** So this is an install-and-configure job, not a config change.

#### ⚠️ New finding — user enumeration is OPEN on live
`/wp-json/wp/v2/users` → **200**, returns the full list:
| id | slug | name |
|---|---|---|
| 2 | `owendeutsch` | Owen Deutsch |
| 3 | `clairegillman` | claire gillman |
| 18 | `clairewisegenstudio-com` | Claire Gillman |

Only **3 accounts total** (page 2 returns `[]`). No `roles` field exposed. `/wp-json/wp/v2/users/2`
also leaks the author archive URL and Gravatar hash.

This matters directly for 2FA: an attacker already has all three **login names**. Combined with no
2FA and no visible login rate-limit on live, the only thing standing between them and wp-admin is
the password. The theme's `kl_block_user_enumeration_attempts()` blocks `?author=N` (403) but does
**not** block the REST endpoint — that is the gap.

**Fix (theme, no account changes):** filter `rest_endpoints` to drop the users route for
non-logged-in requests, or `rest_authentication_errors` to require auth for `/wp/v2/users`.

#### Safe 2FA procedure — zero lockout risk
1. **Backup + WP Engine restore point** (non-negotiable; this is the rollback).
2. Install **Wordfence** on live (staging already runs 9.1.1 — proven).
3. `Wordfence > Login Security > Settings`:
   - Enable **2FA**.
   - **Leave the grace period ON** (default 2 days). Do **not** set 0.
   - **Do NOT** tick "require 2FA" / "enforce for all users" yet.
   - Leave **"Allow remember me"** on so a device is not re-prompted daily.
4. **Enrol one admin at a time.** Have that person scan the QR, then **log out and log back in**
   to prove it works **before** moving to the next account.
5. Only after all three accounts are enrolled, optionally enable enforcement.
6. **Break-glass, always available:** WP Engine portal → SSH → rename
   `wp-content/plugins/wordfence` to disable 2FA instantly. Keep portal access confirmed *before*
   step 3.

**Do not** use "reset all 2FA" or "require 2FA immediately" toggles — those are the lockout paths.

**Accounts to enrol (all 3, none removed):** `owendeutsch` (id 2), `clairegillman` (id 3),
`clairewisegenstudio-com` (id 18).

### functions.php REVIEW (client-supplied, live version) — 2026-11-24

Client pasted the **live** `functions.php`. It differs from the repo zip in two ways:
- **Contains the comment-feed 410 blocker** (`add_action('template_redirect', ...)` + `is_comment_feed()`)
  at the very end. **This is NOT in the repo zip** — repo would revert it on next theme deploy.
- **Contains `CC_Comment_Cleaner`** (the spam-cleanup admin page), also not in the repo zip.
Both must be ported into `owendeutsch.zip` or the next deploy silently undoes them.

#### 🔴 CRITICAL — hardcoded secrets in theme code, committed to git (34 commits)
| Secret | Location | Value (do not paste in reports) |
|---|---|---|
| **SendGrid API key** | `owen_newsletter_user()` — 2 `curl` headers | `SG.RljqRy9qT5mWzJUiGd28MA....` |
| **Google API key** | `my_acf_init()` → `acf_update_setting('google_api_key', ...)` | `AIzaSyAiwA3ml...` |
| **REST API shared secret** | `get_custom_poster()` / `get_poster()` — `md5('hexa@123')` | `hexa@123` (weak, unhashed literal) |

**This is the single most serious finding on the site.** The theme zip is in a git repo
(`mibrahimseofficial-git/DevHub-owendeutsch`). Anyone with repo access has a live SendGrid key
that can send email as the client's domain, plus a Google API key.
**Action:** rotate the SendGrid key immediately (SendGrid dashboard → API Keys → delete + recreate),
restrict the Google key by HTTP referrer + API in Google Cloud Console, move all three to
`wp-config.php` constants or env vars, then **purge the secret from git history** (BFG/filter-repo)
— deleting it in a new commit leaves it in all 34 prior commits.

#### 🔴 CRITICAL — SQL injection, unauthenticated
`odp_autocomplate()` (note the typo in the action name) is registered on **both**
`wp_ajax_autocomplate` and **`wp_ajax_nopriv_autocomplate`**, i.e. reachable by anyone:

```php
$search = $_POST['q'];                       // no sanitisation at all
"... post_title like '%$search%' ..."        // interpolated straight into SQL
```

Same pattern in `odp_search_birds_from_tags_and_keywords()`, `get_id_by_title()`,
`cus_posts_where()`, `addEntry()` (`WHERE email = '".$email."'`), and
`update_save_post_content_callback()`.

**Verified live:** `admin-ajax.php` is reachable (GET 400, POST 200 — an earlier 403 was WAF noise).
A `q=zzzznotabird' OR '1'='1` probe was **blocked by Cloudflare** (403 "Just a moment..."), so the
WAF is currently masking this, but a WAF is a mitigation, not a fix. Do not rely on it.
**Action:** `$wpdb->prepare()` everywhere, or `esc_sql()` at minimum. Highest priority after the
SendGrid key.

#### 🟠 Unauthenticated data exposure — `?test` handler
`if (isset($_GET['test']))` at **global scope** runs a full `WP_Query` over all lodges + all 8,978
birds on **every anonymous request** to any URL with `?test`. Measured earlier: **8.76s, 209 KB**.
It is also a trivial CPU/memory amplification vector. **Delete it** (or wrap in
`current_user_can('manage_options')`).

#### 🟠 Debug output leaking to anonymous visitors
`if (isset($_GET['navnav']))` echoes `<pre>` blocks on the public site. `admin_notices` dumps the
**full taxonomy list** on every admin page (no capability check, no screen check).
`force_post_categ()` and `wpse_59871_script_enqueuer()` also inject admin JS globally.
**Action:** remove or gate behind `current_user_can('manage_options')`.

#### 🟠 Custom REST endpoint with a guessable key
`register_rest_route('api/v5', '/poster', ...)` is public and authenticates on
`md5('hexa@123')`. `get_custom_poster()` returns 250 bird records (title, slug, featured image,
tags, dates). `get_poster()` returns **every attachment ID + GUID** in the database.
`md5()` of a 9-char string is not a secret. **Action:** delete `get_poster`, move the poster
endpoint to a real capability or a long random token stored in `wp-config.php`.

#### 🟡 `cron-requests.log` written into `wp-content/`
`my_log_cron_request()` does `file_put_contents(WP_CONTENT_DIR . '/cron-requests.log', ...)` on
**every cron request** — unbounded growth, no rotation. Currently **403** (blocked), but it is one
`.htaccess`/rule change away from being a public log of every cron URL. **Action:** remove it, or
send to `error_log()`.

#### 🟡 Misc
- `add_filter('posts_where', 'cus_posts_where', 10, 2)` runs on **every** query site-wide, building
  `LIKE '%...%'` chains from `$_GET`. Contributes to the 8.76s `?test` time.
- `add_action('pre_get_posts', 'rlv_manage_queries')` — the function is declared *after* the
  `add_action`, which works but is fragile.
- `classic-editor` is still installed; `wpse_59871_script_enqueuer()` hides the inline-edit UI for
  `cp_birds` via JS rather than a filter.
- `add_filter('widget_text', 'do_shortcode')` is a known XSS amplifier if any widget text is
  editable by a lower-privileged role.

### COMMENT LOCKDOWN VERIFIED ON LIVE — 2026-11-24

Client reports all comment points done. **Re-verified independently — all four pass:**

| # | Check | Result |
|---|---|---|
| 1 | `/comments/feed/` | **410 Gone** — stable across 5 consecutive probes (an earlier single 403 was WAF/edge noise, not the real state) |
| 2 | Recent Comments widget | **GONE** — `/2024/`, `/2024/05/`, `/category/conservation/`, `/blog/` all return `li=0 widget=0` (previously 5 spam items each on date archives) |
| 3 | Comment form on posts | **GONE** — `form=0` on 4 sampled posts. `single.php:29 comments_template()` is commented out, so only an empty `<div class="comment-template">` renders |
| 4 | `wp-comments-post.php` | **403** on both GET and POST — submission endpoint closed |

**Spam comments themselves: no longer visible anywhere.** All three public surfaces (feed, widget,
post pages) are clean. Deletion is complete as far as an anonymous visitor can observe.

**Note on `/2024/05/feed/` → 200:** this is the normal **post** feed (4 blog items from May 2024),
NOT a comment feed. Verified by parsing: `<rss>` root, items are "What's in a Name?",
"Remarkable Bird Eyes", "Coming Home", "Down Indy Dunes". Not a leak.

**Residual cosmetic issue (not a security problem):** every post still emits
`<link rel="alternate" type="application/rss+xml" title="... Comments Feed" href=".../comments/feed/" />`
in the `<head>`. The URL now returns 410, so it is dead, but the dangling link tag remains. Harmless;
can be removed via `remove_action('wp_head','feed_links_extra',3)` if the client wants it clean.

**Still open elsewhere:** staging has NOT had any of this applied (still serves the comment feed and
the old theme), and staging is still on Forminator 1.53.1.

### FORMINTOR UPDATE — LIVE DONE, STAGING LAGGING — 2026-11-24

**LIVE: 1.58.6 ✅** — verified `/entry/` asset URLs now carry `ver=1.58.6` (was 1.53.1).
`1.58.6 >= 1.58.5` patched threshold → **CVE-2026-120676 is CLOSED on live.**

Post-update smoke test on `/entry/` (all pass):
- `forminator-module-31993` present, `data-form-id="31993"`
- nonce field present
- all 4 fields present: `name-1`, `name-2`, `email-1`, `consent-1`
- submit button present
- 4 JS + 4 CSS assets load
- **no** PHP `Fatal error` / `Warning` / `Notice` / `Deprecated` in the rendered HTML
- page HTTP 200

**⚠️ STAGING IS NOW THE EXPOSED ENVIRONMENT: still 1.53.1.** Verified `/entry/` on
`owendeutsc2stg` returns `ver=1.53.1`. The fix went to live first, staging was skipped.
Staging is on `owendeutsch01` and is reachable at a public hostname — it must be updated too.

**⚠️ `readme.txt` is stale.** `wp-content/plugins/forminator/readme.txt` still reports
`Stable tag: 1.53.0` even though the running version is 1.58.6. Do **not** use readme.txt to
determine the installed version on this site — read `?ver=` off the asset URLs instead.
(That is how the earlier "1.53.1" figure was confirmed, and it is the reliable method here.)

**Not verified:** actual form submission. `wp-admin/admin-ajax.php` → 403 (WAF), and posting a
real entry would create a genuine subscriber + notification email. Left to the client to
submit one test entry in a browser.

**Remaining after this:** update staging to 1.58.6, then re-scan to confirm the finding clears
on both environments.

### PHASE 1-2 CLOSE-OUT AUDIT — 2026-11-24 (client approved 5 workstreams)

**HARD BLOCKER: we have NO admin access to either environment.**
`/wp-admin/` and `/wp-login.php` return **403** on live AND staging. No WP-CLI, no SSH/SFTP,
no WP Engine portal credentials, no `wp-json/wp/v2/plugins` (403). Deploy = client drops
`owendeutsch.zip`; DB/plugin/comment work = client in wp-admin. Everything below is either
**repo work (done)** or **an instruction sheet the client must execute**.

#### 1. COMMENTS — "delete all 124,212"
**Number is wrong.** Last measured approved-comment count was **500** (Comment Cleaner run,
493 = 98.6% spam). 124,212 is almost certainly the **total row count of `wp_comments`
including spam/trash**, not approvable comments. Client should confirm in
`wp-admin > Comments > All | Spam | Trash` before deleting, so real comments are not lost.

**Current state (live-verified 2026-11-24):**
- `comments/feed/` → **200**, `application/rss+xml`, **10 items**, newest **16 Jul 2026**.
  NOT disabled yet. (An earlier probe returned 403 — transient, re-test returned 200.)
- Recent Comments widget **still rendered**: `/2024/` → 5 items, `/2024/05/` → 5 items.
  Confirmed spam authors: `Anthony Brown` → `sampletemplatex1234.info`, `创建免费账户`,
  `Binance推荐奖金`, `www.binance.info sign up`, `binance signup`.
- `/category/conservation/` and `/blog/` → 0 (widget only on date archives).
- `single.php:29` `comments_template()` is commented out, so no comments render on posts.

**Exact client steps:**
1. Backup (BlogVault) + WP Engine restore point.
2. `wp-admin > Comments`: filter **All**, empty **Trash**, then bulk-select **Delete Permanently**.
   If >1 page, raise "Screen Options > Number of items per page" to 999.
3. `Settings > Discussion`: **uncheck** "Allow people to submit comments on new posts".
   Then `Posts > All > Screen Options 999 > Bulk Edit > Comments: Do not allow`.
4. `Appearance > Widgets`: remove the **Recent Comments** widget.
5. `Settings > Discussion`: **uncheck** "Allow link notifications from other blogs
   (pingbacks and trackbacks)" — closes the `xmlrpc.php` pingback surface.
6. Keep Akismet active (5.7.2) as a second line.

#### 2. PLUGINS — deletion list
**Verified present on both envs** (`readme.txt` 200): slider-and-carousel-plus-widget-for-instagram,
reorder-by-term, wp-all-export (1.6.0), classic-editor (1.7.0), export-all-urls (6.0),
wp-crontrol (2.0.0), akismet (5.7.2), forminator (1.53.0/1.53.1), wordpress-seo.
Staging additionally: wordfence. Live: no wordfence.

**Active (front-end asset present) — DO NOT DELETE:**
| Plugin | Evidence |
|---|---|
| `instagram-feed` (Smash Balloon) | 6 refs on live homepage |
| `yith-infinite-scrolling` | 2 refs on live + staging |
| `wordpress-seo` (Yoast) | site-wide SEO |
| `akismet` | spam filtering |
| `forminator` | `/entry/` newsletter form (also the CVE — update, do not delete) |
| `slider-and-carousel-plus-widget-for-instagram` | **ACTIVE ON STAGING ONLY** (1 asset ref). 0 refs on live. Client is right. |

**Safe to delete — inactive, zero theme references:**
`reorder-by-term`, `wp-all-export`, `export-all-urls`, `wp-crontrol`, `classic-editor`.

**⚠️ DO NOT DELETE `classic-editor` without a check** — it is harmless but removing it only
matters if an editor relies on the classic editing screen. Confirm with Owen first.

**⚠️ MapSVG — NEVER DELETE.** Not in the probed list (dir name differs) but the theme has
**~18 active `wp_mapsvg_regions_486` / `wp_mapsvg_schema` queries** in `functions.php`
(L2320, L2345 are inside `wp_ajax_nopriv_*` handlers that run for anonymous visitors).
Deleting the plugin risks the tables with it and would break `/globe/` + country status.

**⚠️ `instagram-feed` placeholder images**: live homepage currently shows
`plugins/instagram-feed/img/placeholder.png` — the Instagram feed is failing to load and
falling back to placeholders. Separate issue, worth flagging.

#### 3. USERS / 2FA — "do not remove accounts"
**No account will be touched.** 2FA status:
- **Live: no 2FA plugin at all.** No Wordfence, no two-factor plugin. Live has **zero 2FA**.
- **Staging: Wordfence present** (readme 200). Wordfence 2FA is **per-user and opt-in by
  default**, so enabling it does not lock anyone out.
- `/wp-json/wp/v2/users` → 403, so the user list cannot be enumerated externally.

**Safe 2FA rollout (no lockout):**
1. Install **Wordfence** on live (already proven on staging).
2. `Wordfence > Login Security`: enable 2FA but leave **"Grace period"** on and do **not**
   tick "require 2FA for all users" until each admin has enrolled.
3. Enrol admins **one at a time**, each confirming a successful login before the next.
4. **Keep WP Engine portal access as break-glass** — it can disable a plugin via SSH if a
   user is ever locked out.
5. Do not change any user's role, password, or email.

#### 4. BIRDINGTOP500 BADGE — repo DONE, staging NOT
- Repo `footer.php:101` = hotlink `button.php?u=owendeutschphotography005`, alt "Birding Top 1000",
  `width="207" height="83" loading="lazy" decoding="async"`, no `border` — **committed `daa085f`**.
- **Staging still serves the static PNG**: `.../themes/owendeutsch/images/birdingtop500.png`.
  Confirmed 2026-11-24. Staging theme predates the fix.
- Client instruction "Staging still has the static PNG" — **confirmed correct**.

#### 5. HOMEPAGE IMAGE DIMENSIONS — staging done, live NOT
- Repo `templates/template-homepage.php` has **17 `od_image_size_attrs()` calls** (commit `f17cecd`).
- **Staging: 18 imgs, 6 missing dims** (all UI chrome: bird-loader.gif, logo.png x2,
  loader.svg, birdingtop500.png, close-btn.png). Content images fixed.
- **Live: 22 imgs, 20 missing dims** — content images (perumap, scheduled bird, search-icon,
  owendeutsch.png, BotTA-Graphic, instagram-owl) still have no dimensions. **Fix not deployed.**
- `/grid/`, `/globe/`, `/countries/`, `/blog/` are **identical on both** (4 missing = chrome only).

**⚠️ DO NOT "deploy staging to live".** AGENTS.md line ~771 records that staging **lags live**
on every theme fix (conservation 5 missing, /blog/ 99 missing at last check). A staging→live
DB push would roll back live fixes. Correct path: deploy the **theme zip** from `development`.

#### ORDER OF OPERATIONS
1. Backup both envs.
2. Deploy `owendeutsch.zip` (`development`) to **staging** → verify `/entry/` + homepage dims + badge.
3. Client sign-off.
4. Deploy same zip to **live** → purge WP Rocket → WP Engine → Cloudflare.
5. Forminator update 1.53.1 → 1.58.6 (staging first) — **security, highest priority**.
6. Plugin deletions (after confirming the list above).
7. Comment purge + widget removal + feed/pingback disable.
8. 2FA rollout, one admin at a time.

### WP ENGINE SECURITY ALERT — Forminator 1.53.1 (CVE-2026-120676) — 2026-11-24

WP Engine flagged both environments:
- `owendeutsc2stg` on `owendeutsch01` — **1.53.1**
- `owendeutsch21` (LIVE) — **1.53.1**

**Verified live:** `/entry/` renders Forminator form **#31993** on **1.53.1** (all 7 asset URLs
carry `?ver=1.53.1`). Staging identical (also 1.53.1, readme stable tag 1.53.0).

**Only ONE page uses Forminator.** Crawled all 30 sitemap pages: `/entry/` is the sole page with
Forminator markup (108 refs). Homepage, `/contact/`, `/about-owen/`, `/conservation/`,
`/newsletter/` → **0**. The footer newsletter is **Mailchimp**, not Forminator.

**What `/entry/` is:** a **newsletter signup form** — First Name, Last Name, Email, consent.
**Publicly reachable and unauthenticated.** That matters because the CVE is
"Unauthenticated Stored XSS" — the attack is *submitting the form*, which any visitor can do.

**CVE details (Patchstack, PSID 6d012b79a7c4):**
| Field | Value |
|---|---|
| CVE | CVE-2026-120676 |
| Type | Unauthenticated Stored Cross-Site Scripting |
| CVSS | **7.1** (medium priority) |
| Vulnerable | **<= 1.58.4** |
| **Patched in** | **1.58.5** |
| Required privilege | Unauthenticated |
| User interaction | Required (privileged user must view the entry) |
| Published | 2026-11-20 |
| Researcher | Maciej Komorowski |

**Version gap:** installed 1.53.1 → **latest 1.58.6** (wordpress.org, updated 2026-11-17).
That is **5 minor versions / ~5 releases behind**, not a single patch bump. Do NOT apply just
1.58.5 — go to 1.58.6.

**Why 5 versions behind is risky here:** 1.53.1 is far below the `<= 1.58.4` vulnerable range,
so it is exposed to this CVE **and** every other Forminator issue fixed in 1.54–1.58.5.

**Mitigation already partly in place:** Forminator markup is on exactly one page, so a temporary
deactivation would only break `/entry/` — the newsletter signup. Acceptable stopgap if the update
cannot be done immediately.

**Remediation (staging first, per workflow):**
1. Backup (BlogVault) + WP Engine restore point.
2. Staging: update Forminator 1.53.1 → **1.58.6**.
3. Smoke-test `/entry/`: form renders, all 4 fields present, nonce intact, submit succeeds,
   success message shows, Mailchimp/email handoff still works.
4. Check for PHP deprecations in the log (PHP 8.4.27 — a big jump may surface warnings).
5. Purge WP Rocket → WP Engine → Cloudflare, then re-test (Autoptimize bundles the form CSS;
   a stale bundle can break the form layout/nonce).
6. Promote to live, re-verify `/entry/`.
7. Re-scan Wordfence to confirm the finding clears.

**Gotcha:** Autoptimize serves the form CSS from
`/wp-content/cache/autoptimize/autoptimize_single_*.php`. After the update, clear Autoptimize's
cache too, or the old bundled CSS persists.

**Client-facing note:** recommend enabling WP Engine Smart Plugin Manager or at minimum
Forminator auto-updates — this is the second time drift has created exposure.

### CORRECTION — Birding Top badge IS the "tracking code" (client is right) — 2026-11-24

My earlier note said "no tracking code" in the footer. **Technically true, but misleading.**
The client is correct: `button.php?u=owendeutschphotography005` **is** the tracking mechanism.
Birding Top 1000's own homepage states it is *"the ranking by traffic site for birders"* — the
badge image URL carries the account ID and every fetch is logged server-side as a hit. It is an
image beacon, not a JS pixel: no cookie, no script, no client-side tracking. But removing the
hotlink **removes the site from the ranking**.

**FULL HISTORY of this block (from git, theme zip):**

| Commit | Date | Change |
|---|---|---|
| (original) | — | Commented-out block, **hotlinked** `button.php?u=owendeutschphotography005`, alt "Top 1000" |
| `7e40cec` | 2026-11-03 21:49 | **Task 2: removed the block entirely** (birdingtop500.com was HTTP 000 / 20s timeout) |
| `363bbf6` | 2026-11-16 16:06 | **Restored** — server back online, uncommented, added `target=_blank rel=noopener` |
| `e303751` | 2026-11-16 19:43 | Added `width/height/loading=lazy`, dropped `border` |
| `a1a666f` | 2026-11-16 19:47 | Reverted `e303751` |
| `3b3b2de` | 2026-11-16 20:23 | **Self-hosted** the PNG (`images/birdingtop500.png`), dropped the hotlink |
| **(client)** | ~2026-11-20+ | **Client re-added the hotlink on LIVE** |

**LIVE right now (2026-11-24), exact markup:**
```html
<a href="https://www.birdingtop500.com/" target="_blank" rel="noopener">
  <img src="https://www.birdingtop500.com/button.php?u=owendeutschphotography005"
       alt="Birding Top 500" width="207" height="83" loading="lazy" border="0">
</a>
```

**⚠️ REPO AND LIVE ARE NOW DIVERGED.** Repo `footer.php:101` = self-hosted
`$THEME_PATH/images/birdingtop500.png`. Live = hotlinked `button.php`. Both images exist on live
(the self-hosted PNG still returns 200, 24,199 bytes — leftover from our deploy).
**A theme deploy from this repo would silently revert the client's change.** Repo must be synced
to the hotlinked version before the next deploy.

**Why we self-hosted originally (`3b3b2de`):** button.php is a small hobby server that was
**fully offline for months** (HTTP 000 / 20s timeout in Nov 2025) and still fails intermittently.
When it fails the browser renders alt text. That risk is real and returns with the hotlink.

**Cost of the hotlink:** `cache-control: no-cache, no-store, must-revalidate` + `expires: 0` →
**uncached on every page view**. Across ~2,535 HTML pages that is one third-party request per
view, plus a hard dependency on their server for the footer to render correctly.

**Trade-off for client:** hotlink = counted in the ranking, but slower + fragile.
self-host = fast + reliable, but **not counted**. The client has chosen the ranking.

**Action:**
1. Sync repo `footer.php` to the hotlinked markup so the next deploy does not revert it.
2. Fix `alt` → "Birding Top 1000" (site is "Birding Top 1000", live says "Top 500").
3. Drop deprecated `border="0"` while keeping `width/height/loading=lazy`.
4. Optionally add `decoding="async"`.
5. Tell the client: if the badge ever shows as text, birdingtop500 is down again.

### TRACKING CODE AUDIT — footer / header — 2026-11-24

**Question asked:** is there tracking code in the footer for "Birding Top"?

**Answer: NO tracking code.** The footer block is a plain badge `<a><img></a>`. But the badge is
**hotlinked to a third-party PHP endpoint**, and the theme header carries **four real trackers**.

**FOOTER — `footer.php:99-106` (repo):**
```html
<a href="https://www.birdingtop500.com/" target="_blank" rel="noopener">
  <img src="<?php echo $THEME_PATH; ?>/images/birdingtop500.png" alt="Birding Top 1000 Websites" border="0">
</a>
```
**LIVE differs from repo** — live is hotlinked, not a local file:
```html
<img src="https://www.birdingtop500.com/button.php?u=owendeutschphotography005" alt="Birding Top 500" width="207" height="83" loading="lazy">
```
- `button.php` → **200**, `image/png`, 24,199 bytes, real 207x83 PNG (matches HTML attrs).
- Account ID: **`owendeutschphotography005`**.
- Sends `cache-control: no-cache, no-store, must-revalidate` + `expires: 0` → **never cached**, so
  every page view hits birdingtop500.com. That is a third-party request + a view counter on their
  side, not analytics on ours.
- Alt text mismatch: repo says "Top 1000", live says "Top 500".
- **Recommendation:** download the badge, host locally in the theme, keep the outbound link.

**HEADER — `header.php`, four trackers load on EVERY page (all confirmed reachable 200):**

| Line | Tracker | ID | Note |
|---|---|---|---|
| 20-27 | **GA4** `gtag.js` | `G-KYQ10WWELQ` | current |
| 44-51 | **Universal Analytics** `analytics.js` | `UA-3248496-1` | ⚠️ **dead property** — UA sunset 2023-07-01; sends data to nothing, pure page-weight cost |
| 53-65 | **Facebook Pixel** + noscript img | `723046811532883` | live |
| 67-75 | **LuckyOrange** (`__lo_site_id`) | `220193` | session recording/heatmaps — `d10lpsik1i8c69.cloudfront.net/w.js` 200 |

**Also in header:** `msvalidate.01` Bing verification (line 10) — benign. `oss.maxcdn.com`
html5shiv/respond (lines 14-16) — dead CDN, inside `[if lt IE 9]` so never loads in modern browsers.

**Footer Mailchimp** (`footer.php:20-21, 67-68`): CSS from `cdn-images.mailchimp.com`, JS from
`s3.amazonaws.com/downloads.mailchimp.com/js/mc-validate.js`, form posts to
`owendeutsch.us18.list-manage.com`. This is a newsletter signup, not a tracker.

**Privacy exposure:** GA4 + Facebook Pixel + LuckyOrange (session recording) all fire without a
consent gate. For a site with an EU/UK audience this is a GDPR concern — LuckyOrange records
keystrokes/mouse movement. **Flag to client.**

**Recommended cleanup (theme, staging first):**
1. Remove the UA-3248496-1 block (`header.php:43-51`) — dead, zero loss.
2. Decide on LuckyOrange (`header.php:67-75`) — heavy, and the biggest privacy item.
3. Self-host the Birding Top badge; fix alt text to match live ("Top 500").
4. Consider consent gating for GA4/FB Pixel.

### COMMENT CLEANER RESULTS + SITE-HEALTH DELTA — 2026-11-20

**Comment Cleaner (client-supplied plugin) ran on live.** Sample of 500 random APPROVED
comments: **493/500 were spam (98.6%)**. Remaining "real-looking" ones investigated and are
ALSO spam — see below. Recommendation: delete all + close comments (pending client sign-off).

**"Real-looking" comments are off-topic link-building spam, NOT genuine:**

| Author | Posted on | Body | Verdict |
|---|---|---|---|
| ABM Insulation (x2) | Jabiru: the Giant Stork of Belize | "Your dedication to quality content shows in absolutely everything you create" | generic flattery + commercial link |
| Mike | Picturing the Beauty of Little Brown Jobs | "healthy soil does so much of the work before anything even starts growing... gardening" | **gardening on a bird-photo post** |
| Sarah | What Makes for Great Bird Photography? | "helped a friend streamline her small online shop... order process" | **e-commerce on a bird-photo post** |
| İLKER | Acorn Woodpeckers | "The topic is well chosen, but it lacks examples." | generic filler |
| Alex H. I. Mccall / Hailey Aubrielle Erin Smith | Acorn Woodpeckers | "# Tasarımdan Dijital Pazarlamaya: Sanatizi Reklam Ajansı" | Turkish ad-agency spam |

None reference birds, photography or the post's actual subject. Safe to delete.

**WHERE THE COMMENTS COME FROM (client asked):** bots POST directly to `wp-comments-post.php`
(live, returns 405 on GET). A visible form is NOT required. `comments_open()` is true on all
posts, and `single.php:30` has `//comments_template();` commented out — so the theme stopped
RENDERING comments at some point while the endpoint stayed open. Highest comment ID observed
**314,352** → large historical volume.

**Post-cleanup state still needs 2 actions:** Recent Comments widget still renders spam on
`/2024/`, `/category/*`, `/2024/05/` (5 links), and `/comments/feed/` still serves spam.

**⚠️ CORRECTION: Akismet IS installed** — `wp-content/plugins/akismet/akismet.php` → **200**
(v5.7.2, INACTIVE). My earlier 404 was a transient fetch failure. The doc's "installed but
inactive" was right.

### SITE-HEALTH DELTA vs the audit doc (from client's Site Health export, 2026-11-20)

Several Critical/High items are now RESOLVED — the doc is stale on these:

| # | Audit said | Now (Site Health) | Status |
|---|---|---|---|
| 1 | WP core 7.1.3 → update to 7.1.4 | **7.1.5** | ✅ DONE (ahead of the target) |
| 8 | PHP 8.2.35, upgrade to 8.3 | **PHP 8.4.27** | ✅ DONE (ahead) |
| 10 | WP_MEMORY_LIMIT 40M | **WP_MEMORY_LIMIT 512M / MAX 512M** | ✅ DONE |
| 7 | 12 users, 10 admins | **user_count: 4** | ✅ largely DONE |
| 14 | 6 inactive themes | **1 inactive** (Genesis Block) | ✅ largely DONE |
| 3 | Removed slider plugin active | **now INACTIVE** | ✅ DONE (2026-11-19) |

Still OPEN: #2 Forminator 1.53.1 (still the vulnerable version), #4 auto-updates (only Redirection
enabled of 23), #6 plugin backlog, #9 inactive plugins (**14 now — grew from 9**), #11 OPcache
(`opcode_cache: not available`), #12 spam comments, #13 RUCSS scheduled action.
Also: `WP_DEBUG_DISPLAY: true` while `WP_DEBUG: false` — the display flag should be false.

**⚠️ SECURITY HEADERS ARE FLUCTUATING.** Observed two different header sets on live within
~1 hour:
- earlier: `strict-transport-security`, `x-content-type-options: nosniff`, `referrer-policy:
  strict-origin-when-cross-origin`
- now (8/8 consecutive checks): `x-frame-options: SAMEORIGIN`, `referrer-policy: same-origin`,
  **no HSTS, no nosniff, no Permissions-Policy, no CSP**
Likely mid-change (client was editing `.htaccess`). Do NOT report a fixed header state to the
client until it stabilises. Note WP Engine ignores `.htaccess` anyway.

### MAINT#12 — ~160 spam comments with suspected malware URLs — 2026-11-18

**Doc item:** High / Medium (1-2 h). Wordfence flagged ~160 comments + 1 trackback.

**CONFIRMED: spam is real, live, and publicly visible.** But the doc's "SEO poisoning" framing
is **overstated** — see the nofollow finding below. The real damage is **brand reputation**.

**Where the spam actually appears (verified live, cache-busted):**

| Surface | Exposes spam? | Evidence |
|---|---|---|
| Post pages (`/owl-fun-facts/` etc.) | **NO** | 25/25 sampled posts render **0** comment blocks |
| Archive/category/date pages | **YES — 5 links** | `/2024/`, `/category/uncategorized/`, `/2024/05/` |
| `/comments/feed/` (RSS) | **YES — 10 latest** | full spam author list, links, titles |
| Per-post comment feeds | **YES** | `/wader-wonderland/feed/` → "buy iphone duo with warranty" |
| Homepage / `/blog/` | NO | sidebar not rendered |

**Live spam authors seen in the "Recent Comments" sidebar widget:**
`buy iphone duo with warranty` → `bxnndjmjss.com`; `ABM Insulation` → `abmtrade.eu` (x2);
`aslı` → `aslitarcanglobal.com/hair-transplant-cost-in-turkey/`;
`dr arslan` → `drarslanklinik.com/harmonyca-hibrit-dolgu`; plus `Avukat SEO`, `gartavembkis1984`,
`boshpria`, `yakup çil` in `/comments/feed/`.

**⚠️ GOOD NEWS — the links do NOT pass SEO authority.** Every spam author link carries
`rel="ugc external nofollow"` (checked all 5 in the sidebar; **zero followed links**). WordPress
applies this automatically. So the "SEO poisoning / link equity" risk in the doc is **not
happening**. The exposure is purely reputational: a wildlife-photography brand publicly showing
"buy iphone", "hair transplant turkey", "insulation" links on its archive pages.

**WHY comments never show on posts — theme finding:** `single.php:29-30` is

```php
<div class="comment-template">
<?php //comments_template(); ?>     <- COMMENTED OUT
```

So `wp_list_comments()` / `comment_form()` never run on single posts. `comments.php` still exists
and is correct, but it is never included. Consequence: **no comment form is rendered anywhere**
(`id="commentform"` absent on sampled posts), yet the "Recent Comments" **widget** calls
`get_comments()` directly and happily prints the spam. That is the only reason it is visible.

**Spam can still be submitted:** `wp-comments-post.php` returns **405** on GET, i.e. the endpoint
is live and will accept a POST. With no visible form, submissions are direct/bot POSTs.

**Akismet is NOT installed** — `wp-content/plugins/akismet/akismet.php` → **404**. The doc says
"installed but inactive"; it is actually **absent**. Do not plan around activating it.

**Scale signal:** highest comment ID observed is **314,352**. IDs are sequential, so total
historical comment volume is far larger than the ~160 Wordfence flagged. Expect a big backlog of
pending/spam/trashed rows in `wp_comments`; the 160 are just what Wordfence matched. **Needs a
DB-level count** (`SELECT comment_approved, COUNT(*) FROM wp_comments GROUP BY comment_approved`).

**Remediation (WP Admin / DB — not theme-fixable):**
1. `wp_comments`: `SELECT * FROM wp_comments WHERE comment_approved='spam'` then bulk-delete;
   repeat for `'0'` (pending) and audit `'1'` (approved) — approved ones are what render now.
2. Delete the trackback flagged by Wordfence.
3. **Close comments on old posts** (Settings → Discussion, or bulk edit → Comments: Closed).
   Since `comments_template()` is commented out, closing comments costs nothing visually.
4. **Turn off the Recent Comments widget** (Appearance → Widgets) — instantly removes the public
   spam surface on archive pages. Fastest single win.
5. Install + configure **Akismet or Antispam Bee** (Akismet is absent, not just inactive).
6. Disable comment feeds if unused: Settings → Discussion, or filter `comments/feed`.
7. Re-scan with Wordfence to confirm the count hits zero.

**Repo option (if wanted):** also delete the dead `<div class="comment-template">` block in
`single.php` so the intent is explicit, or restore `comments_template()` if comments should work.

### MAINT#10 — WP_MEMORY_LIMIT 40 MB — 2026-11-18

**Doc item:** High / Low effort (15 min). "Raise WP_MEMORY_LIMIT to 256M (and confirm
WP_MAX_MEMORY_LIMIT stays at 512M) in wp-config.php."

**NOT theme-fixable.** `wp-config.php` is not in this repo (repo holds only `owendeutsch.zip`,
the two client docs, and `docs/`). Theme PHP contains **0** `MEMORY_LIMIT` / `ini_set` /
`memory_limit` references — verified across every `.php` in the zip. Memory is a
wp-config / WP Engine platform setting. **Cannot be verified externally either**: `/wp-json/`
and `/wp-admin/site-health.php` both 403, and no `x-powered-by` is exposed (headers show only
`server: cloudflare`).

**LIVE stress test (cache-busted) — no memory failures at current 40M:**
| page | http | bytes | secs |
|---|---|---|---|
| `/?s=a` (broad search) | 200 | 479,573 | 2.31 |
| `/?s=bird` | 200 | 498,969 | 2.02 |
| `/?s=e` | 200 | 483,419 | — |
| `/globe/` (D3) | 200 | 78,075 | 2.02 |
| `/lodges/` | 200 | 273,407 | 0.61 |
| `/grid/` | 200 | 65,399 | 1.66 |
| `/countries/` | 200 | 78,635 | 1.81 |

Zero `Fatal error` / `Allowed memory size` / `memory exhausted` strings. So 40M is **surviving
front-end traffic today** — the risk is concentrated in **admin/bulk operations**, exactly as
the doc says.

**⚠️ THE REAL MEMORY HOG — found while auditing this item: an unauthenticated global-scope
handler in `functions.php:3122`.**

```php
if (isset($_GET['test'])) {          // line 3122 — NO capability check, NO nonce
    // loads ALL 147 lodges into $lodges_id_title_arr
    // then loads ALL 8,978 cp_birds with posts_per_page => -1
    // then nested foreach: 8,978 x 147 string normalisations + comparisons
}
```

It sits at **file scope in `functions.php`** (not inside a function, not hooked to
`admin_init`), so it executes on **any** front-end request carrying `?test`. Verified live:
`https://owendeutsch.com/?test=1` → **HTTP 200 after 8.76 s** (vs 0.6–2.3 s for normal pages),
performing a full 8,978-record table scan on every hit.

Two problems in one:
1. **Memory/time** — the single heaviest operation in the theme, and it is user-triggerable.
2. **DoS vector** — anonymous, unauthenticated, ~9 s of PHP+DB per request. Easy to saturate
   WP Engine workers.

**Recommendation:** delete the whole `if (isset($_GET['test'])) { ... }` block
(`functions.php:3122-3187`) — it is a one-off migration script that was used to populate
`bird_locations_multi` via `update_post_meta`, and that line is already commented out (3180).
It does nothing except burn CPU. This matters more than the memory bump.

**Other unbounded queries (admin-only, lower risk):**
- `functions.php:3568` — `render_bird_reorder_page()` loads **all** `cp_birds` with
  `posts_per_page => -1` + `get_the_post_thumbnail()` per row → this is the admin page most
  likely to white-screen at 40M. Reorder/Beautiful-Birds admin screen.
- `functions.php:1103, 1124, 1216, 1237` — search helpers, `fields => 'ids'` (lighter, but still
  unbounded over 8,978 rows).
- `functions.php:3272-3403` — batched at `250` (fine).
- `template-lodges.php:84, 126` — `-1` over 147 lodges (fine).

**Action split:**
- **Client / WP Engine (infra):** add `define('WP_MEMORY_LIMIT', '256M');` and confirm
  `define('WP_MAX_MEMORY_LIMIT', '512M');` in `wp-config.php`; then test bulk edit, WP All
  Export, Regenerate Thumbnails. Note WP Engine manages `wp-config.php` — confirm the define
  survives their platform updates / ask support.
- **Repo (theme):** remove the `?test` block; consider bounding the reorder page.

### ABANDONED-PLUGIN AUDIT — "Slider and Carousel Plus Widget for Social Media" (v1.9.3) — 2026-11-18

Maintenance doc Critical #3 says this plugin was removed from wordpress.org but remains installed
and active. **Verified: installed, active, and loading assets site-wide — but its feature is used
NOWHERE.** Safe to deactivate/delete once one dependency is confirmed.

**Evidence — what the plugin emits on live:**
- `assets/css/font-awesome.min.css?ver=1.9.3` — **every page** (homepage, /about-owen/, /faq/,
  /conservation/, /blog/, /globe/, lodges, galleries — 12/12 sampled).
- `iscwp-public-css` (its own stylesheet, `id='iscwp-public-css-css'`) — most pages.
- **NEVER any slider/grid markup.** Zero `iscwp-slider`, `iscwp-grid`, `iscwp-carousel`,
  `wpsisac`, `sbc_`, `instagram-feed-widget` tokens on any page. Literal `[iscwp` string absent
  (shortcode not even present unparsed).

**ROOT CAUSE FOUND (2026-11-18, via String Locator).** Option 817 is
`option_name = options_instagram_feed_shortcode`, `autoload = no` — an **ACF options-page field**
(`options_instagram_feed_shortcode` is the `options_` prefix ACF applies to `get_field($name,
'option')`). Value: `[iscwp-slider username="owendeutsch" limit="8" slidestoshow="4"...]`.

**The theme reads it — but every read is commented out.** Three templates each do:

```php
//$instagram_shortcode = get_field('instagram_feed_shortcode', 'option');
//echo do_shortcode($instagram_shortcode);
echo do_shortcode("[instagram-feed num=5 cols=5 showfollow=false]");   // <- Smash Balloon, hardcoded
```

- `templates/template-homepage.php:953-956`
- `templates/template-about-owen.php:130-132`
- `templates/template-book.php:119-121`

Verified: **zero** uncommented `get_field('instagram_feed_shortcode')` anywhere in the theme.
So the ACF field is written to the DB and then never read. That is the exact reason the abandoned
plugin renders nothing while still enqueuing its CSS — the switch to Smash Balloon was done by
commenting out the old call and hardcoding the new shortcode, leaving the ACF field orphaned.

**This is the cleanest possible removal case:** delete option 817 (and the ACF field) → deactivate
→ delete the plugin. Nothing references either. Of the 5 String Locator hits, 4 are noise — 1 real (`wp_options` 817) plus 3
Wordfence file-index rows (`wp_wffilemods.filename`, `wp_wffilemods.real_path`,
`wp_wfknownfilelist.path/wordpress_path`) that merely record the plugin's own
`includes/shortcode/iscwp-slider.php` file path.

**Why it never renders — verified end to end:**
1. Theme registers exactly ONE widget area, `sidebar` (`functions.php:253`).
2. `get_sidebar()` is called from only two places: `templates/loop-content.php:61` (used by
   `archive.php` + `category.php`) and `templates/template-samplepage.php:20`.
3. Live AND staging both render the sidebar with **only two widgets**: `recent-posts-2` and
   `recent-comments-2`. No slider widget, no text widget holding the shortcode.
4. So option 817 sits in a widget that is not assigned to any active sidebar (e.g. it lives in
   `wp_inactive_widgets`, or in a sidebar that no longer exists). The data is orphaned.

**Verified: zero rendered markup on every page type tested** — 453 sitemap URLs (posts, pages,
lodges) + category archives (`/category/uncategorized/`), date archives (`/2024/`, `/2024/05/`),
`/taxonomy/`, galleries, `/globe/`. Zero `iscwp-slider|grid|carousel|pics|img|wrap|col` tokens,
zero literal `[iscwp-slider` strings in output. Both live and staging identical.

**The Instagram feed users actually see is a DIFFERENT plugin:** Smash Balloon Instagram Feed
(slug `instagram-feed`, v6.16.0, `sbi-styles.min.css`, `sbi-scripts.min.js`). The theme calls it
from `template-homepage.php:956`, `template-about-owen.php:132`, `template-book.php:121` via
`do_shortcode("[instagram-feed num=5 cols=5 showfollow=false]")`. Not the abandoned plugin.

**Widget:** the theme registers ONE sidebar (`functions.php:253`, id `sidebar`) and calls
`get_sidebar()` only from `templates/template-samplepage.php` and `loop-content.php`. The
abandoned plugin's widget is not rendered on any live page.

**⚠️ ONE dependency to confirm before deleting:** the plugin loads its own **Font Awesome 4.x**
(31,000 bytes). The **theme ALSO bundles Font Awesome 4.3.0** at
`fonts/font-icons/font-awesome.css` (28,732 bytes, 200 OK), imported via `fonts/fonts.css`
(`@import 'font-icons/font-awesome.css'`), enqueued as `style-fonts`. The theme's copy defines
`.fa-facebook`, `.fa-search` etc. — so the icons the theme uses (`fa-facebook`, `fa-instagram`,
`fa-search`, `fa-map-marker`, `fa-th-large`, `fa-youtube-play`, `fa-long-arrow-right`) are served
by the THEME, not the plugin. **Deleting the plugin should therefore be visually safe**, but
verify the icons still render on a staging page after deactivation (load-order: if the plugin's FA
were winning the cascade, removal could expose a missing glyph).

**Recommendation:** deactivate on staging → confirm icons + Instagram section render → delete.
Removes an abandoned, unpatched, internet-facing plugin from the update-drift risk list.

**✅ DEACTIVATED — 2026-11-18. VERIFIED SAFE on LIVE.** Post-deactivation check (cache-busted,
4 pages: `/`, `/about-owen/`, both book pages):
- `slider-and-carousel-plus-widget-for-instagram` → **0 refs** (was on every page). Gone.
- `iscwp-public-css` → **0 refs**. Gone.
- **Instagram feed intact:** `instagram-feed/css/sbi-styles.min.css` + `sbi_*` markup still present
  on all 4 pages. Smash Balloon unaffected.
- **Icons intact:** all 7 `fa` icons still render (`fa-facebook`, `fa-instagram`, `fa-search`,
  `fa-light fa-magnifying-glass`, `fa-long-arrow-right`, `fa-youtube-play`, `sbi` carousel icon).
- All 4 pages HTTP 200.

**Font Awesome dependency resolved (this was the one risk):** the theme's FA loads correctly even
though Autoptimize rewrites `@import 'font-icons/font-awesome.css'` to
`@import url(//owendeutsch.com/wp-content/themes/owendeutsch/fonts/font-icons/font-awesome.css)`.
That target is **200, 28,732 bytes, and defines `.fa-facebook`, `.fa-search`, `.fa-instagram`,
`.fa-youtube-play`, `@font-face FontAwesome`**. Webfonts also 200 (`fontawesome-webfont.woff2`,
`.woff`, `FontAwesome.otf`). So the plugin's FA copy was pure redundancy — **the theme was always
the one serving the icons.**

NOTE: staging still shows `wposPlugin=True` (plugin assets still referenced). Deactivation was
applied to LIVE. If staging must mirror live, deactivate there too.

**Remaining cleanup (optional, no user impact):** the 7 commented-out dead lines
(`template-homepage.php:953-955`, `template-about-owen.php:130-131`, `template-book.php:119-120`)
and `wp_options` option 817 (`options_instagram_feed_shortcode`). Plugin files are still on disk
(assets still 200) — the plugin can now be deleted outright.

### EMPTY-HREF SITE SCAN — LIVE — 2026-11-18

Crawled all 453 sitemap URLs (277 posts + 29 pages + 147 lodges) on live. **14 empty
`href=""` across 10 pages.** Classified by comparing against staging (staging = 0 on the
two damaged pages):

**NEW DAMAGE from the link-replacement pass (live only, staging clean) — 5 anchors:**

| Page | Count | What it looks like |
|---|---|---|
| `/huembo-home-of-the-marvelous-spatuletail/` | **4** | `Return to Abra Patricia: …Spreading<a href=""> </a>|<a href=""> American Bird Conservancy (abcbirds.org)</a>` |
| `/the-global-birding-experience-owlet-lodge-at-abra-patricia-in-peru/` | **1** | `…epicenter of a <a href="">culture of conservation</a> spreading…` |

**Cause:** the ABC URLs were blanked (href set to empty) instead of repointed to the Wayback
replacement. The anchor wrapper survived with `href=""`. This is the exact failure mode
AGENTS.md already warns about ("a bare `href=""` is the one outcome to avoid") — same class as
the Mike Parr / Owlet damage. **Fix = set the href to the Wayback URL, or unwrap the `<a>`.**

**PRE-EXISTING theme furniture (identical on live AND staging) — 9 anchors, NOT damage:**

| Page | Anchor text | Nature |
|---|---|---|
| `/first-look-birds-of-the-atlantic-rainforest/` | `Northern (Yellow-bellied)` | `class="whitebtn"` tag with no term URL |
| `/lodges/big-sur-california-usa/` | `Oregon Group` | ditto |
| `/lodges/dixon-illinois-usa/` | `winter plumage` | ditto |
| `/lodges/labagh-woods-illinois-usa/` | `Molting` | ditto |
| `/lodges/mchenry-county-illinois-usa/` | `Molting` | ditto |
| `/lodges/monterey-bay-california-usa/` | `Western gull with a Black Oystercatcher in front` | ditto |
| `/tropical-andes-migration/` | `Molting` | ditto |
| `/which-bird-photography-gear-is-right-for-you/` | (2 × empty text) | adjacent to B&H links; pre-existing per 2026-07-16 Wayback |

To tell damage from furniture: **furniture is byte-identical on staging; damage appears only on
live.** Use live-vs-staging diff, not the count alone.

### DEAD-LINK REPLACEMENT VERIFICATION — LIVE — 2026-11-18

User reported the 14 live dead links were replaced. Re-crawled live (424 pages: 277 posts +
147 lodges) → 265 unique external URLs → tested all. **12 of 15 old URLs confirmed gone;
3 still present** (see below). Two of the 3 are the *known-unfixable* class.

| Old URL still rendered on live | Page | Note |
|---|---|---|
| `doi.org/10.1675/1524-4695(2006)29%5B169:WCADBI%5D2.0.CO;2)` | `/its-gone-but-where-did-it-go-an-aba-young-birder-essay/` | trailing `)` **not** removed |
| `http://hummingbirdscanada.ca/hummingbirds/migration` | `/chip-cold-and-canada-an-aba-young-birder-essay/` | agreed REMOVE — still present |
| `amazon.com/…/dp/1787135047?tag=smarturl-20\uFEFF` | `/greater-flamingo-the-dramatic-migrator/` | trailing **U+FEFF** still present |

**CONFIRMED FIXED on live (12):** all 7 ABC Birds article URLs, `conserva-aves.org/en/home-english/`,
`seattle-audubon-nature-shop…/search/BirdNote/` (×2 pages), `birdsnotmosquitoes.org/frequently-asked-questions`,
`chicagobirder.org/cos-labagh-woods-project`→`/labagh`, `ecoanperu.org/indexeng.html`, `felzfotoz.com/portfolio-1/portfolio-italy`,
`marylandbiodiversity.com/docs/engstrom_2023.pdf.` (trailing dot gone), `visittoledo.org/birding-in-northwest-ohio/`,
and **Roku** — repointed to a NEW valid channel id (`9046d648…:13639b48…`, verified 200).

**Lesson (repeat of OA-2):** the 3 stragglers are invisible-character / punctuation edits. A
"search & replace" that looks successful in the editor can still miss a trailing BOM/`)`/`.`
because the character is not visible. Verify by **byte-level comparison of the rendered href**,
not by eyeballing the field. Also: WP Rocket + Cloudflare cache by full query string — use a
fresh random `?cb=` per request or you audit the stale page.

**STAGING post-update health (2026-11-18)** — user updated WordPress core, theme and all
plugins on staging. Smoke-tested 12 key pages (`/`, `/blog/`, `/videos/`, `/globe/`, `/grid/`,
`/countries/`, `/conservation/`, `/lodges/`, `/?s=birds`, `/my-recent-photos/`, `/about-owen/`,
`/llms.txt`): **all HTTP 200, exactly 1 H1, 1 JSON-LD, 1 jQuery** each. No regression. The
`/?s=birds` page's 207 `no-dimension` images are **UI chrome** (logo.png, loader.svg,
bird-loader.gif, search-icon.png, musical-notes-symbols-2.png) — identical counts on live, so
**not** an update regression. `/llms.txt` 200 (theme handler).

## Client document task status
Work in CLIENT DOCUMENT order. Statuses: DONE (staging) / OPEN / FALSE POSITIVE.
**As of 2026-11-12 the client deployed the theme directly to LIVE** — every "DONE (repo/staging)"
theme fix below was re-verified in production; see **LIVE VERIFICATION SWEEP — 2026-11-12**.
Task numbering follows the user's "PDF#N" labels (31 tasks). Note the PDF's slide
"03 · 7 Relative Lodge URLs + 44 Dead External Links" is split into TWO client tasks:
PDF#3 = relative lodge URLs, PDF#4 = the 44 dead external links.

| # | Task | Status |
|---|------|--------|
| PDF#1 | 8,424 broken location links (HTTP 400) | DONE (staging) — Part B false positive |
| PDF#2 | Dead birdingtop500 footer badge | DONE (staging) — repo synced |
| PDF#3 | 7 relative lodge URLs missing scheme | DONE (staging) — commit `0558a7e` |
| PDF#4 | 44 dead external links | **DONE (2026-11-12)** — all 8 actionable links fixed and verified live (6 repointed, 2 removed as domains with no successor); see the broken-links section below. Original analysis: VERIFIED (2026-11-02) — **NOT THEME-FIXABLE**. Zero of the dead links are in theme code (grep of all 148 theme files: none of nanpa/owletlodge/africanpenguinsafe/nikon/abcbirds/seattle-audubon/roku/lincolnpark/orangehill/sandpiperabaco/sanisidrolodge/zebrahills/lemalacamp/wildsumaco/birdingtop500 appear). They live in **post content** (blog articles) and **ACF lodge fields** (database). Fix = client edits content / Redirection plugin, not a theme deploy. Live re-test 2026-11-02 shows many have RECOVERED since the Oct-2026 crawl: nanpa.org 200, nikonusa.com 200, lincolnparkconservancy.org 200, orangehill.com 200, sandpiperabaco.com 200, sanisidrolodge.com 200, zebrahills.com 200. Still genuinely broken: birdingtop500.com (server dead — badge already removed PDF#2), owletlodge.org (timeout), africanpenguinsafe.org (530), 5x abcbirds.org article links on /huembo.../ + 2 ABC pages (site restructured /blog20//article/ -> /news/), seattle-audubon shop (404), channelstore.roku.com detail (400), lemalacamp.com + wildsumaco.com (timeout). Footer Zoom webinar link (June 2020, 404) is inside a COMMENTED-OUT modal — dead code, never rendered. |
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

### LIVE VERIFICATION SWEEP — 2026-11-12 (client deployed theme straight to LIVE)

The client replaced the theme **directly on live** (skipping the staging step). Every
theme fix below had previously been verified on **staging only** — so this sweep confirms
the whole theme deliverable is now actually running in production. Method: cache-busted
fetches with a **fresh random** param each time (`?v=<rand>`, `x-cache: MISS`).

**Confirmed LIVE — all PASS:**

| Task | Check | Live result |
|------|-------|-------------|
| PDF#1 | Location filter links (were HTTP 400 via `%FF`) | All previously-broken locations **200**; `%FF` corruption = **0**; proper en-dash links render (5 on san-diego) |
| PDF#2 | birdingtop500 badge | **0** refs on homepage |
| PDF#3 | Relative lodge URLs (`/lodges/.../www.`) | **0** on san-isidro and san-diego |
| PDF#5 | W3Schools keywords meta | **0** |
| PDF#6 | Viewport zoom block | `user-scalable=0` **absent** |
| PDF#7 | Cache-Control/Pragma metas | **0** |
| PDF#8 | `/cp_birds_country/ec/` + `/lodges/southeast-california/` | Both valid: DOCTYPE/html/head/body/title present, ec = 76 KB with 1 H1 (not 566 bytes); 0 script/style after `</html>` |
| PDF#9 | Filter case/space normalization | `flight` / `FLIGHT` / `" flight"` all **200** |
| PDF#10 | `/llms.txt` | **200**, `text/plain`, 1525 bytes |
| PDF#11 | Duplicate JSON-LD | **1** block, `@vocab` gone |
| PDF#12 | Missing H1s | `/globe/`, `/grid/`, `/cp_birds_country/us/` each have an H1; homepage = 1 H1 |
| PDF#15 | Duplicate jQuery | **1** core `jquery.min.js` (jquery-migrate + jquery.validate are plugins, not duplicates) |
| PDF#27 | `/globe/` external CDN scripts | **0** (all self-hosted) |

**Two false FAILs in my own check code (fixed, not real defects):** (a) I counted
`jquery.validate.min.js` as a duplicate core jQuery — it is a plugin; the real core count
is 1. (b) I lower-cased the page then searched for the uppercase literal `'<!DOCTYPE'`,
which can never match — ec page does have a valid DOCTYPE. Lesson: assert on the specific
thing (core jQuery, case-insensitive DOCTYPE), not a loose substring.

**Still client/infra (unchanged, not theme):** PDF#13 WebP (Cloudflare/Imagify), PDF#14
security headers (edge), PDF#25 HTML minify (WP Rocket setting), PDF#19/#20/#21
(content/Yoast), PDF#31 (process).

### FULL CLIENT-PDF RE-VERIFICATION — every task vs LIVE — 2026-11-12

Pulled the client PDF again and re-checked **all 31 tasks** against the live site
(cache-busted, fresh random param each request). Result: **every theme-code task passes;
the remaining gaps are all content/infra (client-owned).**

**PDF#1–#3, #5–#12, #15, #27 — ALL PASS on live.** (location filters 200 with 0 `%FF`,
no birdingtop500, no relative-www lodge URLs, no keywords meta, no ISO-8859-1 charset,
viewport zooms, no cache metas, ec + southeast-california valid, /?s= noindex, /blog/page/2/ 200,
llms.txt 200, single JSON-LD, all 14 country/globe/grid H1s present, 1 core jQuery,
/globe/ self-hosted.)

**Recovered on their own (no action needed):** nanpa.org 200, nikonusa.com FTZ 200,
lincolnparkconservancy 200, orangehill / sandpiperabaco / sanisidrolodge / zebrahills /
lemalacamps / wildsumaco all 200.

**Changes since the last check (2026-11-12):**
- **robots.txt `Crawl-delay: 10`** — **still present** (re-checked; NOT removed). See OPEN list.
- **Security headers — PARTIAL, improved.** Now present: `Strict-Transport-Security`
  (`max-age=31536000`), `X-Content-Type-Options: nosniff`, `Referrer-Policy:
  strict-origin-when-cross-origin`. Still absent: `X-Frame-Options`, `Content-Security-Policy`,
  `Permissions-Policy`.

**HTML payload also much smaller than the audit claimed:** homepage **208 KB** (audit said 607 KB),
activity search **~0.5 MB** (audit said 1.76 MB). **Still unminified** (`rocket-minify` absent,
148 lines, no leading-indent lines → WP Rocket HTML minify still OFF).

#### STILL OPEN — genuinely broken external links (content, client-owned)

A full sitewide scan (455 pages: posts + lodges + pages; 274 unique external links) found these
**genuinely broken** (404 / DNS / 5xx / timeout). Bot-blocked statuses (401/402/403/429 — B&H,
Cornell, TripAdvisor, DOI, Unsplash, USA Today) are **not** defects.

| Broken URL | On page(s) | Fix |
|---|---|---|
| `abcbirds.org/get-involved/bird-smart-glass/` | /backyard-bird-photography-mike-parrs-lockdown-therapy/ | site restructured → repoint to abcbirds.org/news/ |
| `abcbirds.org/program/hawaii/take-action/` | /the-mosquito-line-blurs-…-hawaiis-birds/ | ditto |
| `abcbirds.org/article/complementary-actions-protect-both-rare-birds-…/` | /huembo-home-of-the-marvelous-spatuletail/ | ditto |
| `abcbirds.org/article/peruvian-postage-stamps-feature-endangered-birds/` | /huembo…/ | ditto |
| `abcbirds.org/blog20/bird-conservation-superstars/` | /huembo…/ | ditto |
| `abcbirds.org/it-takes-a-village-to-save-rare-hummingbirds/` | /huembo…/ | ditto |
| `abcbirds.org/return-to-abra-patricia-a-culture-of-conservation-is-spreading/` | /huembo…/, /the-global-birding-experience-owlet-lodge-at-abra-patricia-in-peru/ | ditto |
| `seattle-audubon-nature-shop.shoplightspeed.com/search/BirdNote/` | /acorn-woodpeckers-…/, /double-crested-cormorant-a-birdnote-story/ | shop closed → link birdnote.org |
| `channelstore.roku.com/details/…/owen-deutschs-beautiful-birds` | /learning-about-birds-with-the-beautiful-birds-screensaver/ | channel page gone (timeout) → channelstore.roku.com root |
| `www.africanpenguinsafe.org/` | /african-bird-focus-south-africa/ | **530** (origin down) — remove or repoint |
| `www.ecoanperu.org/indexeng.html` | /abc-mike-parr-my-favorite-bird/ | 404 → root ecoanperu.org is 200 |
| `bahamasgeotourism.com/entries/dream-villas/…` | /the-bahamas-birds/ | **503** (whole domain down) |
| `conserva-aves.org/en/home-english/` | /bird-tourism-in-the-tropical-andes-means-hope/ | 404 → conserva-aves.org root 200 |
| `www.chicagobirder.org/cos-labagh-woods-project` | /surprise-broad-billed-hummingbird-in-chicago/ | 404 → root 200 |
| `www.visittoledo.org/birding-in-northwest-ohio/` | /spring-migration-magic-in-may/ | 404 → root 200 |
| `www.birdsnotmosquitoes.org/frequently-asked-questions` | /saving-the-forest-birds-of-hawaii/ | 404 → root 200 |
| `www.felzfotoz.com/portfolio-1/portfolio-italy` | /lessons-from-a-lens/ | 404 → root 200 |
| `www.marylandbiodiversity.com/docs/engstrom_2023.pdf.` | /morning-flight-in-appalachia-…/ | **trailing dot typo** — remove the `.` |
| `hummingbirdscanada.ca/hummingbirds/migration` | /chip-cold-and-canada-…/ | DNS fails |
| `femalebirdsong.org/` | /the-female-of-the-species/ | DNS/connection fails |
| `www.kws.go.ke/parks/nairobi-national-park` | /the-best-lens-and-camera-for-bird-photography/ | connection fails |
| `amazon.com/…/dp/1787135047?tag=smarturl-20\ufeff` | /greater-flamingo-the-dramatic-migrator/ | **invisible U+FEFF** appended to URL — strip it |

**Also found (not in the PDF, plugin-level):** `popup-maker/dist/packages/block-library-style.css`
**404s** on every page (confirmed still live; first seen in the client's log export).

#### STILL OPEN — client/infra (not theme code)

| Item | Status on live |
|---|---|
| PDF#9 robots.txt `Crawl-delay: 10` | **STILL PRESENT** (not removed) |
| PDF#14/#30 security headers | PARTIAL — HSTS + nosniff + Referrer-Policy present; **X-Frame-Options, CSP, Permissions-Policy absent** |
| PDF#25 HTML minify | **Still OFF** (unminified; payload already 0.2–0.5 MB) |
| PDF#13/#24 WebP | Not enabled (99.9% JPEG) |
| PDF#13/#23 empty alt + missing dims | **Gallery images DONE + DEPLOYED TO LIVE (2026-11-11, commit `1fcfd74`)** — `template-conservation.php` L50 and `index.php` L32 were the two gallery templates missed by the original PDF#13 pass; both now emit width/height via `od_image_size_attrs()` / the `$image[1..2]` guard, matching template-globe.php L216 and grid_template.php L104. Remaining no-dimension images are **UI chrome only** (loader.svg, logo.png, birdingtop500.png, close-btn.png, musical-notes-symbols-2.png, bird-loader.gif) plus content-inserted images (WP core adds dims automatically).  **LIVE-VERIFIED after deploy:** `/conservation/` 5/5 gallery images + `/blog/` 100/100 thumbnails now carry width/height; `/grid/`, `/globe/`, `/countries/` 30/30 each. **Staging still runs the pre-fix theme** (conservation 5 missing, /blog/ 99 missing) — staging lags live on every theme fix; never push staging->live. |
| PDF#11 Yoast Knowledge Graph | No Organization node yet (client must configure Yoast) |
| PDF#20/#21 duplicate titles/H1s | Sample of 200: **0 duplicate titles**, 3 duplicate H1 pairs (content-level) |
| PDF#31 monthly audit | Process, not code |

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

**Baseline vs damage (do not misread the sitewide scan).** Final verified state (2026-11-12, post-deploy): **53 pages** show `href=""`, and **all 53 are pre-existing theme furniture**, not damage:
- **State/region buttons** on lodge pages (`Alaska`, `Illinois`, `Ohio`… — 5 each) and the tag/related-post widgets on blog posts (`Clear`, `Molting`). The state buttons are `<a class="whitebtn" href="">State</a>`; confirmed present in the 2026-03-07 Wayback snapshot of `/lodges/maumee-ohio-usa/`.
- **`/lodges/dixon-illinois-usa/`** — 5 state buttons + `<a class="whitebtn" href="">winter plumage</a>` (a **Plumage** tag with no term URL). Pre-existing: the 2026-05-21 Wayback snapshot already had 6 `href=""` and 4 "winter plumage".
- **`/which-bird-photography-gear-is-right-for-you/`** — 2 empty anchors `<a href=""></a>` (no text) next to the B&H product links. Pre-existing: the 2026-07-16 Wayback snapshot already had exactly 2.
- **`/monterey-bay-california-usa/`** — one caption-ish anchor.

To tell damage from furniture, look at the anchor TEXT: `Ohio`/`Illinois`/`Clear` = baseline. The only genuine damage found this cycle was Mike Parr (`The Owlet Lodge`), since fixed.

**Cache-busting lesson (cost me a false alarm):** WP Engine + Cloudflare cache aggressively by **full query string**. Reusing one param value all session (e.g. `?z=1`) returns a **stale cached page** (`x-cache: HIT` / `cf-cache-status: HIT`) even after a deploy. Always use a **fresh random** query value per request when verifying, or you will audit the old theme and think the deploy failed.

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

**FIXED + DEPLOYED + VERIFIED (2026-11-12) — live.** Both copies now read:

```php
<?php the_field('credit_name'); ?>
<?php if(get_field('credit_url')): ?></a><?php endif; ?>
```

so the close tag is emitted only when the open tag is. Rendering verified by simulation: with a URL -> `<a href="…" target="_blank">© Name</a>` (unchanged); with empty `credit_url` -> `<h3 class="desktop-show">© Name</h3>` (no orphan tag). `php -l` clean, all 43 theme PHP files lint clean, zip rebuilt with 150 entries / order preserved / `testzip()` clean / only `single-lodges.php` differs. Client replaced the theme directly on **live**.

**Live verification (fresh cache-busting params):** stray credit `</a>` sitewide = **0** (was 52). `/lodges/abaco-bahamas/` credit now `<h3 class="desktop-show">© Owen Deutsch</h3>`, `href=""` = 0. `/lodges/cabanas-san-isidro-lodge-ecuador/` credit still renders as a link (`<a href="https://www.sanisidrolodge.com/" …>…</a>`) — no regression. Pages with `href=""` = 53, **all pre-existing** (see next note).

**Note:** `single-lodges-bk.php` carries the same bug but is a stale backup (no `od_normalize_url`, not a WP template name so never loaded) — deliberately left untouched.

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

**Removed from the repo (2026-11-12):** all generated progress/verification reports were
deleted from `development` at the client's request — the repo now carries only the two
source audit documents (see below). The reports remain retrievable from git history
(e.g. `git show <commit>:Owen-Deutsch-Audit-Progress-Report.docx > file.docx`).

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
Report (removed from repo 2026-11-12): `Owen-Deutsch-Security-Headers-Addendum-2026-11-06.docx`.

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
