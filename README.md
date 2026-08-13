
# IndexFixer — Google Index Monitoring for WordPress

**[IndexFixer Pro](https://zinkwp.com/indexfixer.html)** is a WordPress plugin that shows the real Google indexing status of every URL on your site — inside wp-admin, with history — and runs a dynamic internal-linking loop that helps Google discover the pages it keeps missing.

This repository is the public technical documentation. The plugin itself is a paid product (€50/year, single site, [14-day refund](https://zinkwp.com/terms-of-service.html)).

## What it does

- Connects to **Google Search Console** with a **read-only** OAuth scope (`webmasters.readonly`) — no Google Cloud project, no API keys, no JSON files. Two clicks.
- Inspections run on our servers (api.zinkwp.com) via the official **URL Inspection API**, within Google's limits (2,000 inspections/day per property, 600/min). Results are stored server-side and mirrored into your WordPress dashboard.
- Every URL gets a status: **Indexed / Not indexed / Discovered / Excluded**, with verdict, coverage state, last Googlebot crawl time and full history charts.
- New posts are inspected within minutes of publishing, everything else on a rolling cycle with prioritization.

## The indexing loop (the interesting part)

A front-end widget lists your **not-indexed posts as real, visible HTML links** — ordered by neglect:

```
today:            every page  →  /article-8472/    (Googlebot has never crawled it)
tomorrow:         every page  →  /article-9211/    (not crawled for 94 days)
after it indexes: every page  →  /article-10482/   (next in line)
```

1. The widget surfaces what's missing (only live, published pages; never drafts or deleted URLs).
2. Googlebot follows the links — internal links are Google's primary discovery mechanism.
3. IndexFixer moves those exact URLs to the front of **its own inspection queue** (not Google's — nobody controls that).
4. The moment a page turns *indexed*, it drops out and the next one slides in. Fully automatic.

There is also a category-aware variant that only lists not-indexed posts from the current category, keeping links topically relevant.

### What it can and cannot do

**Crawling ≠ indexing.** The loop removes the *discovery* bottleneck ("Google can't find this URL or won't revisit it"). It cannot fix "Google crawled this page and judged it thin or duplicate" — that's a content problem, and no internal link solves it. IndexFixer tells you which case each page is:

- `Discovered — currently not indexed` → discovery / crawl-budget problem → the widget helps.
- `Crawled — currently not indexed` → usually content quality → improve the page first.

The URL Inspection API **only reads status — it never submits anything to Google.** Any tool claiming to "force" indexing of regular pages via an API is misrepresenting what Google allows (the Indexing API is restricted to job postings and broadcast events).

## Numbers from our own sites

Across 12 production sites (data through August 2026, both groups measured on the same clock — from publication date):

| Metric | URLs surfaced by the widget | Comparable URLs without the widget |
|---|---|---|
| Not-indexed → indexed | **68%** (2,278 / 3,329) | 41% (3,350 / 8,257) |
| Median days from publication to indexed | **35** | 38 |
| Indexed within 30 days of publication | **42%** | 38% |

The conversion gap is the headline. On speed, the honest picture: **once a URL enters the widget, half are indexed within 3 days** (median; 59% within a week) — and these are pages that had already been waiting for weeks. That figure uses a different clock (widget entry) and is deliberately not compared to the control group.

Bias note: the widget receives the *most neglected* URLs (never crawled, or not crawled the longest), i.e. the hardest cases — the comparison is stacked against it, not for it. Full methodology and a single-site deep dive: **[case study — a new site from 0 to 98.6% indexed in 5 months](https://zinkwp.com/case-study.html)**.

## Architecture & data

```
Google Search Console ──(read-only OAuth)──► api.zinkwp.com ──(mirror)──► your wp-admin
```

- OAuth tokens encrypted at rest (AES-256-GCM), EU hosting (OVH), TLS everywhere.
- The plugin stores results in a dedicated MySQL table (`wp_indexfixer_urls`) — no `wp_options` bloat, tested with thousands of URLs.
- Read-only by design: IndexFixer cannot change anything on your site or in your Search Console property.
- [Privacy Policy](https://zinkwp.com/privacy-policy.html) · [Data Processing Agreement](https://zinkwp.com/dpa.html) · [Terms](https://zinkwp.com/terms-of-service.html)

## FAQ

**Do I need API keys or a Google Cloud project?**
No. Authorization is a two-click OAuth flow; all API plumbing lives on our servers.

**What GSC permission level do I need?**
Owner or full user of the property. Google's URL Inspection API does not work for restricted users.

**Does it slow my site down?**
No — all checks run on our servers, not yours.

**Is there a URL limit?**
No. Note Google's own inspection cap (2,000/day per property): a 20,000-URL site takes ~10 days per full refresh cycle, which is exactly why widget URLs and fresh posts are prioritized.

**Will the widget get every page indexed?**
No — and an honest tool should say so. See "What it can and cannot do" above.

**Who builds this?**
[Paweł Zinkiewicz](https://zinkiewicz.pl/) — 20 years in SEO, co-owner of the [if.pl](https://if.pl/) agency, publishing at [bynajmniej.pl](https://bynajmniej.pl/) and the [Vibe SEO podcast](https://vibeseo.pl/). IndexFixer is built and battle-tested on his own network of content sites first.

---

**Get it:** https://zinkwp.com/indexfixer.html · **Docs:** https://zinkwp.com/docs.html · **Contact:** hello@zinkwp.com
