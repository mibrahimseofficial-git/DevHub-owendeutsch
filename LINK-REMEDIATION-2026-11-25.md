# Link Remediation — Measured Findings

**Site:** owendeutsch.com · **Date:** 2026-11-25 · **Method:** crawled 357 of 457 sitemap URLs
(108 of 147 lodge pages), extracted 1,818 raw hrefs, probed 300 unique external links following
redirect chains, then verified each finding individually against live.

**Client's brief:** 3 one-character typos · 9 rebranded/moved links · 4 permanently dead sites ·
11 removed partner pages.

**What measurement found:** 4 typos · 33 moved links · 1 dead site · 0 verifiable partner removals.

---

## 1. One-character defects — found 4, not 3

| # | Page | Defect | Impact |
|---|---|---|---|
| 1 | `/why-birds-migrate/` | `href` contains the book **title**, not a URL | **Link is completely broken.** Browser resolves it as a relative path and fails |
| 2 | `/its-gone-but-where-did-it-go-an-aba-young-birder-essay/` | DOI href has a stray trailing `)` | **404** |
| 3 | `/greater-flamingo-the-dramatic-migrator/` | U+FEFF byte-order mark inside the Amazon affiliate tag | Page loads but the tag is unrecognised — **affiliate commission lost** |
| 4 | `/2021-a-year-in-review/` | Internal href `surprise-broad-billedhummingbird-%20in-chicago/)` — space and `)` inside the slug | Reaches the right page only via a redirect |

### Fixes

1. `/why-birds-migrate/` → replace href with the intended Amazon or publisher URL
2. `/its-gone-…/` → drop the final `)` → `https://doi.org/10.1675/1524-4695(2006)29%5B169:WCADBI%5D2.0.CO;2`
3. `/greater-flamingo-…/` → retype the tag as `?tag=smarturl-20`
4. `/2021-a-year-in-review/` → `https://owendeutsch.com/surprise-broad-billed-hummingbird-in-chicago/`

### Reported as typos but are not defects

- `/how-birds-experience-the-fourth-of-july/` — `&amp;` in href is **correct HTML**; returns 200
- `/how-bird-friendly-coffee-helps-bird-conservation/` — same; si.edu returns 403 only to non-browser clients

Both carry a trailing `.` after a `#:~:text=` fragment. Harmless, but worth tidying.

---

## 2. Moved / rebranded links — found 33, not 9

All 33 currently resolve, because every one is still 301/302-redirected. They are not broken;
they are one hop slower and several land somewhere less useful than intended.

**Highest priority — lands on a generic page, not the article:**

| Page | Old link | Now lands on |
|---|---|---|
| `/birds-in-bloom/` | `abcbirds.org/tropicalandesbook` | ABC donate page (not the book) |
| `/backyard-bird-photography-mike-parrs-lockdown-therapy/` | `abcbirds.org/specialoffer/` | ABC donate page |
| `/lodges/belmond-khwai-river-lodge-at-moremi-reserve-botswana/` | Belmond Khwai lodge | `belmond.com/en/lodges` — generic index |
| `/lodges/kwetsani-camp-botswana/` | Kwetsani camp | `wildernessdestinations.com/…/okavango-delta` — region, not the camp |
| `/tapichalaca-reserve-casa-simpson-lodge/` | NeoBird8 PDF | `neotropicalbirdingandconservation.org` home |
| `/blown-away/` | `ebird.org/alerts` | `ebird.org/home` — alerts feature gone |
| `/buenaventura-…/` + `/tapichalaca-…/` | `jocotoco.org/wb#/EN/…` | `jocotoco.org/es/quienes-somos/` — Spanish about page |
| `/morning-flight-…/` | SORA NAB PDF | `digitalcommons.usf.edu/sora/` — journal home, PDF gone |

**Host genuinely rebranded (update the href):**

| Old host | New host |
|---|---|
| `birdcast.info` | `birdcast.org` |
| `rowman.com` | `bloomsbury.com` |
| `globalforestwatch.org` | `globalnaturewatch.org` |
| `chicagotonight.wttw.com` | `news.wttw.com` |
| `neotropicalbirdclub.org` | `neotropicalbirdingandconservation.org` |
| `ak.audubon.org` | `audubon.org/alaska` |
| `wilderness-safaris.com` | `wildernessdestinations.com` (all 3 camps) |

**Path restructured on same host (update the href):**

`reconnectwithnature.org` · `birdsrio.com.br` · `blog.mybirdbuddy.com` · `rspb.org.uk` ·
`birdnote.org` · `abcbirds.org/bird/` → `/birds/` (×2) · `chicagotribune.com` ·
`nikonusa.com` · `channelstore.roku.com` · `sula.com.co` · `jocotoursecuador.com/destination/` → `/`
(×3)

**Cosmetic only, no action needed:** `eBird.Org` (capitalisation), `georgiawildlife.com` (adds
`:443`), `en.m.wikipedia.org` → `en.wikipedia.org` (mobile), Amazon dp → `/clp/` (their own
behaviour).

---

## 3. Permanently dead sites — 1 of 4 is actually dead

| Domain | Client's claim | Measured | Verdict |
|---|---|---|---|
| `africanpenguinsafe.org` | dead | **NXDOMAIN** — no DNS record | ✅ **genuinely dead** |
| `dunesnationalpark.org` | dead | **Live** — now a **handball-betting spam site** | ❌ not dead |
| `beyourownbirder.com` | dead | **Live** — publishing; newest post 23 Nov 2026 | ❌ not dead |
| `lincolnparkconservancy.org` | dead | **Live** — North Pond page present and current | ❌ not dead |

### What to actually do

**`africanpenguinsafe.org`** — remove the link on `/african-bird-focus-south-africa/`, anchor text
"here". Convert to plain text.

**`dunesnationalpark.org`** — **more urgent than the client realises.** The domain did not die, it
was **taken over and repurposed as gambling spam**. Owen's site is currently sending readers —
and passing SEO equity — to a betting site from
`/lodges/indiana-dunes-national-park-indiana-usa/`, anchor "Indiana Dunes National Park – Indiana,
USA". This is a reputational and SEO liability. **Remove the link now.** The correct destination,
if a link is wanted, is `nps.gov/indu`.

**`beyourownbirder.com`** — nothing to do. Still an active birding blog by Melissa Mayntz. Removing
this link would break a legitimate partner reference.

**`lincolnparkconservancy.org`** — nothing to do. Page is live. Anchor "lincolnparkconservancy.org"
on `/lodges/north-pond-illinois-usa/` is fine as-is.

---

## 4. The 11 removed partner pages — cannot be produced

**No such list exists** in `AGENTS.md`, the SEO audit, the maintenance review, or any of the 35
commits. Nothing in the repository records 11 partner pages being removed.

### What the lodge pages actually show

- **147** lodge pages exist; **108** were crawled
- **27** render a partner link inside the `<h1>`
  - **20** point to a real external partner
  - **7** point back to owendeutsch.com itself (Arizona, Gloucestershire, LaBagh Woods, Montana,
    North Texas Coast, Texas Hill Country, Wales) — these are self-links, not partners
- **81** render a bare `<h1>` with no link at all

Of the 20 real partner links, **19 return 200**. One returns 403 to automated clients
(`kagerasafaris.com`, Akagera) — that is bot-blocking, not a dead site.

### Why 81 bare `<h1>`s is not the answer

Those lodges were **never linked**. Their ACF `site_url` field is empty. A page that was never
linked is a different thing from a partner page that was removed — and the client asked for
*removed*, so sending the 81 would answer a question nobody asked.

### To produce the real list

The client's figure of 11 must come from somewhere — most likely their own crawl or a diff against
an older snapshot. Ask for that export, and it can be reconciled exactly. The definitive check is
the ACF `site_url` field per lodge: any lodge with a value in the database but no link in the
rendered page was a link that was removed or broken.

---

## Coverage caveat

357 of 457 sitemap URLs fetched (78%); 108 of 147 lodge pages. BlogVault and Cloudflare throttle
this IP, so counts are a **floor, not a ceiling**. The 100 unfetched URLs are almost entirely
blog posts; a complete pass would need a slower crawl or an allowlisted IP.
