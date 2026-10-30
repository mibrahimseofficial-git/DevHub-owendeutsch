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
**Every task must be performed on STAGING first, verified there, then deployed to LIVE.**
Because the site runs WP Rocket + WP Engine Object/Page Cache + Cloudflare Edge,
every change must trigger a coordinated cache purge:
WP Rocket -> WP Engine -> Cloudflare Edge.

## Source documents (repo: mibrahimseofficial-git/DevHub-owendeutsch, branch main)
- `Owen-Deutsch-SEO-Audit-Presentation-Final.pdf` — the 16-page client audit (31 tasks).
- `Owen-Deutsch-Website-Maintenance-Site-Health-Review.docx` — maintenance/site-health review (21 issues).

## Deliverable conventions
- Client-style documents: navy (#142A45) + orange (#D96B1E), Calibri, footer
  "Owen Deutsch Photography · ... | Page N", prioritised roadmap at the end.
- Generate .docx with python-docx; .pdf with pypdf/pdflatex as needed.

## Questionnaire workflow (how the user wants tasks handled)
One task at a time. For each task:
1. State the task as a question; give live-verified evidence.
2. User verifies: real task or false positive.
3. If real -> provide the solution (staging-first).
4. User verifies / applies, says "done" -> move to next task.

## Verification notes / corrections to the PDF
- **Task 1 (8,424 broken location links):** FALSE POSITIVE as diagnosed. Site emits
  raw UTF-8 (en-dash E2 80 93), not %FF. `strtolower()` cannot produce %FF. The %FF is a
  crawler artifact from the DUAL charset declaration (UTF-8 + ISO-8859-1). Real fix =
  percent-encode the location query + remove the conflicting charset meta. Fix anyway
  because the client re-runs the crawler.
- **Task 2 (birdingtop500 footer badge):** badge markup is ALREADY commented out in
  footer.php — no live link, no image request. Still a crawler artifact; remove the
  commented block.

## Known live-verified facts
- HTTP header sends `charset=UTF-8`; page also declares `<meta charset="UTF-8">` AND
  legacy `<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">`.
- W3Schools placeholder keywords meta present on every page.
- `%FF` appears 0 times in emitted HTML; raw 0xFF byte in query string -> 400.
- Correctly-encoded location URL returns 200 (full 1.7MB results page).
- No security headers (HSTS/X-Frame-Options/nosniff/CSP) on any page.
- debug.log, xmlrpc.php -> 403; REST user enum -> 401. (Good.)
- Plugin readme.txt files publicly readable (200).
