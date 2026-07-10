# Monthly Reports Workflow — A-Life Medica

This document explains how to generate the monthly performance report for **BIOS Line** and **LYSI**, and how to publish it on the live A-Life Reports site.

> **Live site:** https://bios-line-april-2026-report.vercel.app/
> **GitHub repo:** https://github.com/diell-dev/bios-line-april-2026-report
> **Contact:** diell@polarbearagency.com

---

## What you need before you start

The two brands behave very differently — collect data accordingly.

### BIOS Line
- **Facebook Page:** connected to Meta Business Manager. Pull FB stats from there.
- **Instagram (@biosline_kosova):** **NOW CONNECTED to Business Manager (as of June 2026)** — pull IG stats from Insights with the toggle on Instagram, same as LYSI. No more mobile-app screenshots needed unless you specifically want app-level metrics.
- **Ads:** Facebook Ads Manager exports (xls/csv). ⚠️ **Filter by BIOS Line campaigns before exporting** — the ad account contains other clients (Taverna Tirona, Diora, etc.) and unfiltered exports create noise. Use the "First 4" campaign naming convention for the main product-messages campaign.

### LYSI
- **Facebook Page:** connected (small audience, ~528 followers). Skip or footnote.
- **Instagram (@lysikosova):** connected to Business Manager — **pull everything from there**.
- **Ads:** Facebook Ads Manager exports (xls/csv).

---

## What to upload to Claude each month

Drop these files into the conversation when starting a new month:

### 1. Ad campaign exports (both brands)
- From Meta Ads Manager → filter by campaign → date range = the reporting month → Export as `.csv` or `.xls`
- Naming pattern: `<brand>-Campaigns-<start>-<end>.csv`

### 2. Meta Business Suite Insights screenshots — LYSI
Open business.facebook.com → Insights → switch the top-right toggle to **Instagram** → screenshot:
- **Results** tab (Views / Reach / Content interactions / Link clicks / Visits / Follows)
- **Audience → Demographics** (age + gender + top cities)
- **Messaging** (Total contacts, Conversations started, Organic vs Paid breakdown)

### 3. Meta Business Suite Insights screenshots — BIOS Line (Facebook side only)
Same as LYSI but with toggle on **Facebook**:
- Results tab
- Audience → Demographics
- Messaging
- Optional: Content tab (top performing posts)

### 4. Instagram mobile app screenshots — BIOS Line (optional, no longer required)
**As of June 2026, BIOS Line's IG is connected to Meta Business Manager**, so IG stats come from the same Business Suite screenshots as LYSI (Results / Audience / Messaging tabs with the toggle on Instagram). The mobile-app screenshots below are only needed if you want app-specific metrics (like Reels engagement breakdown that Business Suite doesn't show in the same way):
- **Views** screen · **Content by content type** · **Content list** · **Profile activity**

> Historical note: prior to June 2026 these mobile-app screenshots were the only way to get BIOS Line IG data. May 2026 and earlier reports use those; June 2026+ uses Business Suite.

---

## How Claude builds each report

### Step 1 — Pull Instagram post-level data via Apify
Each brand's Instagram public posts are scraped using `apify/instagram-post-scraper`. Known handles:
- BIOS Line → `biosline_kosova`
- LYSI → `lysikosova`

Apify input:
```json
{
  "username": ["<handle>"],
  "resultsLimit": 60,
  "onlyPostsNewerThan": "<YYYY-MM-01 of reporting month>",
  "skipPinnedPosts": false,
  "dataDetailLevel": "basicData"
}
```
Output gives: post URL, type, caption, like/comment counts, timestamps, video views, thumbnail URLs.

Cost: ~$0.05 per scrape.

### Step 2 — Combine all sources
Apify gives the public post grid + visible engagement.
Insights screenshots fill in the metrics that aren't public: reach, impressions, demographics, conversations.
Ad CSVs give campaign-level performance.

### Step 3 — Generate the report
Each brand gets its own `index.html` page in:
```
2026/<month>/<brand>/index.html
2026/<month>/<brand>/thumbs/*.jpg          # IG post thumbnails downloaded from CDN
2026/<month>/<brand>/chart_<X>_orange.png  # matplotlib charts (cost per result, audience age)
```

The pages share a visual language (cream / orange palette, Anton + Inter fonts) but have separate sections that match each brand's emphasis:
- BIOS Line stats slide → Facebook + a smaller Instagram callout
- LYSI stats slide → Instagram (the dominant channel)

### Step 4 — Update the navigation tree
Three index pages handle the year/month/client picker:
- `/index.html` → year picker
- `/2026/index.html` → month picker (add new month tile when adding a new month)
- `/2026/<month>/index.html` → brand picker (BIOS Line + LYSI cards)

When adding a new month, copy the previous month's folder + change content. Update both the year-picker and the new month's index.

### Step 5 — Push to GitHub
Vercel auto-deploys from the `main` branch.

```bash
# from a clean clone:
git -c user.email='diell@polarbearagency.com' -c user.name='Diell Grazhdani' \
    commit -m '<month> 2026: BIOS Line + LYSI reports' && \
    git push origin main
```

The PAT pattern is documented in Diell's private CLAUDE.md (the workflow is: paste fresh PAT into chat each session, Claude uses it as an env var only, you rotate after).

---

## Folder structure

```
/                                       # year picker (links to /2026/, /2027/, ...)
├── README.md
├── vercel.json
├── INSTRUCTIONS.md                     # this file
├── index.html
└── 2026/
    ├── index.html                      # month picker
    ├── april/
    │   ├── index.html                  # brand picker (BIOS Line + LYSI cards)
    │   ├── bios-line/
    │   │   ├── index.html              # the BIOS Line report
    │   │   ├── chart_age_orange.png    # audience age chart
    │   │   ├── chart_cpr_orange.png    # cost-per-result chart
    │   │   └── thumbs/*.jpg            # 11 IG post thumbnails
    │   └── lysi/
    │       ├── index.html              # the LYSI report
    │       ├── chart_lysi_age.png
    │       ├── chart_lysi_cpr.png
    │       └── thumbs/*.jpg            # 14 IG post thumbnails
    ├── may/
    │   └── ...                         # repeat structure
    └── ...
```

---

## Visual design — what stays consistent across reports

- Background: `#F4ECDE` (cream)
- Primary accent: `#E8401E` (orange)
- Display font: Anton (Google Fonts) — Impact-style headers
- Body font: Inter (Google Fonts)
- Decorative arc: orange circle outline at edges of hero/thank-you sections
- Stats band: orange background with white oval bubbles for the dominant channel
- Secondary channel callout: cream background, smaller orange ovals
- Each report ends with a "Thank You" hero + Polar Bear Agency badge + contact

---

## Common issues + fixes

### IG handle not found by Apify
LYSI uses `lysikosova` (no underscore). BIOS Line uses `biosline_kosova` (with underscore). If a new brand is added, run a few candidate handles in one Apify call to discover the right one — wrong ones return `not_found`, the right one returns posts.

### IG Reels show no like count
By default Instagram hides like counts on Reels for non-account-owners. Apify's `instagram-post-scraper` returns `likesCount: 0` or just video views. For Reel-heavy reports, prefer view counts (which are public).

### BIOS Line IG stats (June 2026 onward)
BIOS Line's IG is now connected to Meta Business Manager. Pull all IG stats from Business Suite Insights with the top-right toggle on Instagram (same as LYSI). If you're doing a backfill for a period before June 2026, you'll still need mobile-app screenshots.

### BIOS ad exports contain multiple clients
The ad account also runs Taverna Tirona, Diora, Gjakova KCGF, etc. Always filter Ads Manager by campaign name / label = "BIOS Line" before exporting, or manually filter the raw CSV/xls afterwards. In May 2026 only "First 4" was BIOS Line — the rest were other clients.

### Vercel build fails
The site is pure static HTML — there's no build step. If a deploy fails, the cause is almost always: file too large, broken HTML/CSS, or invalid `vercel.json`. Check the Vercel deploy logs — they're linked from the GitHub commit page.

### GitHub PAT rejected
PATs expire. If push returns 401/403, regenerate at github.com/settings/tokens (fine-grained, scope: Contents read/write on the repo) and paste the fresh one in chat. **Never commit a PAT to the repo.**

---

## Key data points to lead each report with

These are the questions every report should answer in the first 10 seconds of skimming:

1. **What was the total ad spend?** Headline number on the Ad Performance section.
2. **What was the cost per main result?** (Conversation, profile visit, etc — depends on campaign objective.)
3. **Which ad set / campaign was the top performer?** Include € per result.
4. **Which ad set / campaign should be paused or rebuilt?** Be willing to recommend pausing.
5. **Which organic post got the most engagement?**
6. **Did any influencer collab outperform brand-owned content?** This is huge for LYSI; emerging for BIOS Line.
7. **Where is the audience geographically and demographically?** Tirana vs Pristina is critical for LYSI/BIOS distinction.

---

## Versioning

When you add a new month, **never edit a published month's report** — leave history intact. A-Life can compare months by clicking back through the navigation.

Last updated: 2026-06-01 (May 2026 published; June 2026 workflow adjusted for BIOS Line IG now connected to Business Manager).

## Update — June 2026: Meta Ads via connector

As of June 2026, ad campaign data can be pulled directly via the Meta Ads MCP connector — no more CSV exports needed. The connector accesses ad-level performance for all Meta-connected ad accounts.

- **LYSI - A-life** account id: `1463293032171303` (owned by lysikosova business)
- **Diell Grazhdani** personal account id: `31418707` (contains BIOS Line + other clients; filter needed)

For BIOS Line: query campaigns where `amount_spent > 0` and manually identify BIOS Line campaigns by name pattern (typically "First 4"). Non-BIOS ones in this account: Taverna Tirona ("Shank", "Kerkojme…"), Gjakova KCGF, promotional page boosts.

Organic Insights (Views/Reach/Interactions/Demographics/Messaging) still require Business Suite screenshots — the connector is Ads-only.

