# Cloudflare DNS baseline — owendeutsch.com

Snapshot of the Cloudflare DNS zone, captured 6 November 2026.
Use this to diff later when proxy status or records change.

Zone: `owendeutsch.com` (Cloudflare Free)
Nameservers: `marek.ns.cloudflare.com`, `peaches.ns.cloudflare.com`
Records used: 26 of 1,000
DNS Setup: Full

## Web / hosting

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| A | `*.owendeutsch.com` | 172.67.152.112 | **Proxied** | Auto |
| A | `owendeutsch.com` | 141.193.213.10 | **DNS only** | 1 hr |
| A | `owendeutsch.com` | 141.193.213.11 | **DNS only** | 1 hr |
| CNAME | `www.owendeutsch.com` | wp.wpenginepowered.com | **DNS only** | 1 hr |

`141.193.213.0/24` is registered to WPEngine, Inc. (ARIN RDAP: handle `NET-141-193-213-0-1`),
and is **not** in Cloudflare's published IPv4 ranges. The explicit DNS-only records for the
apex and `www` override the proxied wildcard, so the main site bypasses this Cloudflare zone
and is fronted by WP Engine's own edge instead.

## Microsoft 365 — email and identity

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| MX | `owendeutsch.com` | owendeutsch-com.mail.protection.outlook.com | DNS only | Auto |
| CNAME | `autodiscover.owendeutsch.com` | autodiscover.outlook.com | DNS only | Auto |
| CNAME | `mail.owendeutsch.com` | outlook.office365.com | **Proxied** | Auto |
| CNAME | `msoid.owendeutsch.com` | clientconfig.microsoftonline-p.net | **Proxied** | Auto |
| CNAME | `enterpriseenrollment.owendeutsch.com` | enterpriseenrollment.manage.microsoft.com | DNS only | Auto |
| CNAME | `enterpriseregistration.owendeutsch.com` | enterpriseregistration.windows.net | DNS only | Auto |
| CNAME | `lyncdiscover.owendeutsch.com` | webdir.online.lync.com | DNS only | Auto |
| CNAME | `sip.owendeutsch.com` | sipdir.online.lync.com | DNS only | Auto |
| CNAME | `selector1._domainkey.owendeutsch.com` | selector1-owendeutsch-com._domainkey.owendeutsch.onmicrosoft.com | DNS only | Auto |
| CNAME | `selector2._domainkey.owendeutsch.com` | selector2-owendeutsch-com._domainkey.owendeutsch.onmicrosoft.com | DNS only | Auto |
| SRV | `_sipfederationtls._tcp.owendeutsch.com` | 1 5061 sipfed.online.lync.com | DNS only | Auto |
| SRV | `_sip._tls.owendeutsch.com` | 1 443 sipdir.online.lync.com | DNS only | Auto |

`mail`, `msoid` and `_domainconnect` are proxied. This is unusual but verified working:
`mail` returns 301 to `/mail/` with Exchange headers intact, and `msoid` returns 200 from
Microsoft. Mail delivery uses the MX record, which is DNS only and unaffected.

## MailChimp (DKIM)

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| CNAME | `k2._domainkey.owendeutsch.com` | dkim2.mcsv.net | DNS only | 1 hr |
| CNAME | `k3._domainkey.owendeutsch.com` | dkim3.mcsv.net | DNS only | 1 hr |

## GoDaddy / domain services

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| CNAME | `_domainconnect.owendeutsch.com` | _domainconnect.gd.domaincontrol.com | **Proxied** | Auto |

## Verification and email policy (TXT)

| Type | Name | Content | Proxy | TTL |
|---|---|---|---|---|
| TXT | `owendeutsch.com` | `v=spf1 include:spf.protection.outlook.com ip4:108.160.196.92 -all` | DNS only | Auto |
| TXT | `owendeutsch.com` | `facebook-domain-verification=pyppdls40ntt947i2jkie6zzyd643z` | DNS only | Auto |
| TXT | `owendeutsch.com` | `google-site-verification=85xB-K9EOUj_j1-siOJV27ike26H-1oxRb9D4Oi9eNQ` | DNS only | Auto |
| TXT | `owendeutsch.com` | `ca3-6cf5d55e7bf849d4b4913e4b654055e1` | DNS only | Auto |
| TXT | `owendeutsch.com` | `ca3-334468fadd2f4af581f3e7ffa73b2d2d` | DNS only | Auto |
| TXT | `owendeutsch.com` | `MS=ms92629495` | DNS only | Auto |
| TXT | `_dmarc.owendeutsch.com` | `v=DMARC1; p=none;` | DNS only | Auto |

## Totals

- 26 records
- 4 proxied: `*` (wildcard), `mail`, `msoid`, `_domainconnect`
- 22 DNS only

## Open items recorded against this baseline

1. **DMARC is `p=none`** — monitoring only, spoofed mail is not blocked. Move to
   `p=quarantine`, then `p=reject`, after reviewing aggregate reports.
2. **The wildcard `*.owendeutsch.com` is proxied** — every undefined subdomain resolves to
   Cloudflare rather than failing. Harmless, but it makes `includeSubDomains` HSTS riskier
   than it appears. Review whether the wildcard is still needed.
3. **The apex and `www` are DNS only.** This is deliberate and unchanged: the main site is
   fronted by WP Engine's own edge, so response headers for the main domain are set in the
   WP Engine portal (Web Rules > Header Rules), not here. Do not orange-cloud these without
   first confirming with WP Engine whether Global Edge Security is active — double-proxying
   causes redirect loops and cache conflicts.
