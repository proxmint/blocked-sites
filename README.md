# Which top sites block datacenter IPs

**22 of 684 sites (3%)** refuse a request from a
datacenter IP and serve the identical request from a residential one. Measured daily,
last run **2026-09-18 07:36 UTC**.

Every host in the Tranco top 1,000 is asked for its homepage **twice at the same
moment** — once from a datacenter IP, once through a residential exit — and only the
cases where the two answers *disagree* are published here. A site that refuses both
legs is blocking the request, not the IP, so it is recorded as not singling out
datacenter traffic rather than counted as a block.

## The funnel, not just the rate

| | |
|---|---|
| Names ranked by Tranco | 1,000 |
| Never dialled — no address at the apex | **229 (23%)** |
| Serve a homepage | 771 |
| Dialled, no clean answer | 87 |
| **Conclusive** | **684** |
| Refused datacenter, served residential | **22** |
| Refused residential, served datacenter | 23 |

### Why the top 1,000 is not 1,000 websites

Tranco ranks by DNS query volume, not by visitors, so **229 of these names have no
address at their apex at all** — nameserver and CDN domains that answer billions of
lookups and serve a homepage to nobody.

| Rank | Name | Why it was not asked |
|---|---|---|
| 7 | `akamai.net` | unresolvable |
| 14 | `ezviz7.com` | unresolvable |
| 19 | `domaincontrol.com` | private dns |
| 23 | `akamaiedge.net` | unresolvable |
| 25 | `gtld-servers.net` | unresolvable |
| 26 | `akadns.net` | unresolvable |
| 27 | `hicloudcam.com` | unresolvable |
| 33 | `apple-dns.net` | unresolvable |

Full list: [`data/skipped-names.csv`](data/skipped-names.csv).

This is not a footnote. A name with no address of its own still draws an answer from a
residential exit whose resolver replies regardless, and that reads exactly like "the
datacenter request was refused and the residential one succeeded". Resolving every name
first is what keeps 229 infrastructure domains out of the block count.

## Rank barely predicts it

| Band | Block rate | |
|---|---|---|
| Top 100 | 3% | 2 of 73 |
| 101–500 | 2% | 5 of 260 |
| 501–1,000 | 4% | 15 of 351 |

## Who is in front of the refusal

| Edge | Blockers |
|---|---|
| cloudflare | 8 |
| undisclosed | 5 |
| fastly | 5 |
| cloudfront | 2 |
| akamai | 2 |

"undisclosed" means the host announced no CDN in its response headers — not that it has none.

## The list

| Rank | Site | From datacenter | From residential | Edge |
|---|---|---|---|---|
| 36 | `fastly.net` | 403 | 200 | fastly |
| 47 | `digicert.com` | 403 | 200 | fastly |
| 109 | `reddit.com` | 403 | 200 | — |
| 226 | `mit.edu` | 403 | 200 | — |
| 306 | `weibo.com` | no response | 200 | — |
| 319 | `wiley.com` | 403 | 200 | cloudflare |
| 403 | `espn.com` | 202 | 200 | cloudfront |
| 512 | `behance.net` | 403 | 200 | fastly |
| 583 | `teamviewer.com` | 403 | 200 | cloudflare |
| 622 | `patreon.com` | 403 | 200 | cloudflare |
| 634 | `doctolib.fr` | 403 | 200 | cloudflare |
| 636 | `ikea.com` | 403 | 200 | cloudflare |
| 697 | `imgur.com` | 429 | 200 | fastly |
| 717 | `att.com` | 403 | 200 | akamai |
| 740 | `mlb.com` | 403 | 200 | fastly |
| 765 | `mediafire.com` | 403 | 200 | cloudflare |
| 837 | `odoo.com` | 403 | 200 | — |
| 862 | `amazon.com.br` | 202 | 200 | cloudfront |
| 885 | `att.net` | 403 | 200 | — |
| 899 | `kleinanzeigen.de` | 403 | 200 | akamai |
| 943 | `genius.com` | 403 | 200 | cloudflare |
| 959 | `chaturbate.com` | 403 | 200 | cloudflare |

Full set: [`data/blocked-sites.csv`](data/blocked-sites.csv) · [`data/blocked-sites.json`](data/blocked-sites.json)

Currently refusing datacenter IPs: `fastly.net`, `digicert.com`, `reddit.com`, `mit.edu`, `weibo.com`, `wiley.com`, `espn.com`, `behance.net`, `teamviewer.com`, `patreon.com`, `doctolib.fr`, `ikea.com`.

## Files

| File | Contents |
|---|---|
| [`data/blocked-sites.csv`](data/blocked-sites.csv) | 22 blockers with both statuses and the edge |
| [`data/blocked-sites.json`](data/blocked-sites.json) | the same, plus the full funnel, rank bands and edge counts |
| [`data/skipped-names.csv`](data/skipped-names.csv) | the 229 names with no homepage to ask |

```bash
curl -s https://raw.githubusercontent.com/proxmint/blocked-sites/main/data/blocked-sites.csv
```

Live API, same data, with filters — no key, CORS open:

```bash
curl 'https://proxmint.com/api/site-blocks'
curl 'https://proxmint.com/api/site-blocks?format=csv'
curl 'https://proxmint.com/api/site-blocks?blocked=all'
```

## Method

1. **Take the list.** The Tranco daily top 1,000, re-fetched every run so the sample is reproducible.
2. **Resolve first.** Tranco ranks by DNS traffic, so nothing without an apex address is dialled at all.
3. **Ask twice, at the same moment.** A plain `GET /` from our datacenter IP and the same request through a residential exit, following up to three redirects. Status and headers only — never the page body.
4. **Confirm before counting.** Every refusal is re-asked: twice from the datacenter, where the IP never changes, and up to three times residentially, where each attempt draws a different exit. A difference that will not reproduce is not published.
5. **Publish only the disagreement.**

### Limits

- Homepages only. No logged-in pages, no search endpoints, no APIs.
- One datacenter provider and one residential pool. A different pair would move the number; the direction should hold.
- A snapshot per host, not a rate over time. A soft block that fires on the second request looks like a pass here.
- A bot wall that returns `200` with a challenge body counts as served, so **3% is a floor, not a ceiling**.
- The crawler identifies itself as `ProxmintBench/1.0` rather than impersonating a browser. A browser user-agent with realistic Accept headers changed no status on either leg.

## Licence

CC BY 4.0 — <https://creativecommons.org/licenses/by/4.0/>. Use it anywhere, including in something
that concludes you do not need what we sell; credit [Proxmint](https://proxmint.com/free-proxies/blocked-sites).

Built by [Proxmint](https://proxmint.com), who sell proxies. That is why the measurement exists,
and it is why the method and every exclusion are published alongside the number.
Write-up with the daily figures: <https://proxmint.com/free-proxies/blocked-sites>
