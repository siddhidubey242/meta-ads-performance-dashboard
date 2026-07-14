# Meta Ads Performance Dashboard (Power BI)

Interactive Power BI dashboard analyzing Meta (Facebook & Instagram) ad campaign performance across the full funnel — impressions → clicks → purchases — with audience, geographic, and time-based breakdowns.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-217346?style=flat&logo=microsoft-excel&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=flat)

---

## 📊 Dashboard Preview

![Dashboard Overview](screenshots/01-overview.png)

---

## 1. Project Overview

This project analyzes ad campaign performance across Facebook and Instagram, using a synthetic-but-realistic dataset of 400,000+ user-ad interaction events, 200 ad creatives, 50 campaigns, and ~9,800 users. The goal was to build a single-view Power BI dashboard that lets a marketing or growth stakeholder answer three questions at a glance: **which platform and ad format perform best, who is engaging, and where is the funnel leaking.**

I built the full pipeline myself: loading and typing raw CSVs in Power Query, modeling a star schema (one fact table, three dimension tables), writing 11 DAX measures for funnel and cost metrics, and designing an interactive report with a dynamic-measure parameter so a single set of charts can switch between Impressions, Clicks, or Purchases without duplicating visuals.

## 2. Business Problem

A marketing team running paid campaigns across Facebook and Instagram has visibility into raw platform data, but no unified view to answer:
- Which platform and ad format actually convert, not just attract clicks?
- Where in the funnel (impression → click → purchase) is the biggest drop-off?
- Which audience segments (age, gender, geography) should get more budget?
- When should ads be scheduled for maximum engagement?

This dashboard consolidates that into one interactive report instead of manual cross-platform exports.

## 3. Dataset Description

Four related tables, structured as a star schema with `ad_events` as the fact table:

| Table | Rows | Description |
|---|---|---|
| `ad_events` | ~400,000 | Every user-ad interaction: impression, click, share, comment, or purchase, with timestamp |
| `ads` | 200 | Ad creative metadata: platform, format (video/image/carousel/story), and targeting criteria |
| `campaigns` | 50 | Campaign name, start/end date, and budget |
| `users` | ~9,840 | User demographics: age, gender, country, location, interests |

**Relationships:** `ad_events` → `ads` (via `ad_id`), `ads` → `campaigns` (via `campaign_id`), `ad_events` → `users` (via `user_id`).

## 4. Data Preparation Process

All transformations were done in Power Query before loading into the model:

1. **Loaded** four raw CSVs (`ad_events.csv`, `ads.csv`, `campaigns.csv`, `users.csv`).
2. **Promoted headers** on each table (`Table.PromoteHeaders`).
3. **Corrected data types** explicitly for every column — IDs as integers, `timestamp` as datetime, `start_date`/`end_date` as date, `total_budget` as decimal — rather than relying on Power BI's auto-detection, which frequently misreads mixed-format columns.
4. **Built the relationship model**, connecting the fact table to all three dimension tables on their respective keys.
5. **Wrote 11 DAX measures** covering funnel metrics (Impressions, Clicks, Shares, Comments, Purchases, Engagements) and derived rates (CTR, Engagement Rate, Conversion Rate, Purchase Rate, Total Budget, Avg Budget per Campaign), all using `DIVIDE()` with a zero-fallback to avoid divide-by-zero errors.

### ⚠️ Known Data Quality Notes

Two issues surfaced during QA that I'm documenting transparently rather than glossing over:

- **Corrupted user IDs (~2.3% of users):** 225 of 9,841 `user_id` values in the raw `users.csv` were silently converted into scientific notation (e.g. `1.20E+01`) by Excel at some point in the file's history — a well-known Excel behavior when a text ID happens to resemble a number. This breaks the join to `ad_events` for those users, meaning a small slice of engagement data isn't attributed to a demographic profile.
- **Default report view is scoped to Facebook + a single month:** The version of the dashboard reflected in the screenshots below reflects a Facebook-only, single-month (June) filter state, not the full two-platform, four-month dataset. The relative patterns (audience skew, ad format ranking, funnel shape) hold at full scale, but absolute totals in a full refresh would be roughly 4-5x larger across both platforms.

I'm flagging both here rather than hiding them — in a real analytics role, catching and disclosing data quality issues matters more than presenting artificially clean numbers.

## 5. KPIs & Metrics

| KPI | Formula | Purpose |
|---|---|---|
| Impressions | Count of `event_type = Impression` | Measures reach |
| Clicks | Count of `event_type = Click` | Measures engagement intent |
| CTR | Clicks ÷ Impressions | Ad creative/targeting effectiveness |
| Engagement Rate | (Clicks + Shares + Comments) ÷ Impressions | Overall content appeal |
| Conversion Rate | Purchases ÷ Clicks | Funnel efficiency from click to purchase |
| Purchase Rate | Purchases ÷ Impressions | End-to-end funnel efficiency |
| Total Budget | Sum of campaign budgets | Cost basis |
| Avg Budget per Campaign | Total Budget ÷ Campaign Count | Budget distribution |

## 6. Dashboard Walkthrough

The report is a single, comprehensive view combining:
- **Top-line KPI cards** — Impressions, Clicks, Shares, Comments, Purchases, Engagements, and all four rate metrics, plus budget figures
- **Clicks by Gender** (donut) and **Clicks by Age** (bar) — audience composition
- **Clicks by Country** (map) — geographic distribution
- **Weekly and Hourly Click Trends** — time-based engagement patterns
- **Analysis by Month** (calendar heat map) — day-level activity
- **Analysis by Ad Type** (table) — CTR, Conversion Rate, Purchase Rate, and Engagement Rate broken down by Video, Stories, Image, and Carousel
- A **dynamic measure selector** — lets the viewer switch several visuals between Impressions, Clicks, and other metrics without needing separate pages

## 7. Key Insights

*(Figures below reflect the Facebook + June 2025 view shown in the dashboard screenshot; see the data quality note above.)*

- **Strong top-of-funnel, weak bottom-of-funnel:** CTR of 11.35% and Engagement Rate of 13.13% indicate ad creative and targeting are landing well. But Conversion Rate (5.44%) and Purchase Rate (0.62%) show most engaged users never complete a purchase — the funnel leaks hardest between click and purchase, not between impression and click.
- **Audience skews young and female:** Female users account for 43% of clicks vs. 22% male, and engagement is concentrated in the 20–30 age range, tapering off sharply past 35.
- **Video and Stories outperform static formats:** Video leads on Purchase Rate (0.66%) and Conversion Rate (5.56%); Stories has the largest volume (23.8K impressions) with a strong Purchase Rate (0.64%). Image and Carousel trail on both engagement and conversion efficiency.
- **Geographic reach is concentrated in a handful of markets:** the underlying user base is heavily weighted toward the US, UK, Canada, India, and Germany — a mix of high-volume and higher-purchasing-power markets that likely warrant different campaign strategies.

## 8. Business Recommendations

1. **Prioritize conversion-stage fixes over awareness spend.** Since CTR and engagement are already strong, incremental ad spend on more impressions will have diminishing returns. Landing page experience, offer relevance, and retargeting are the higher-leverage fix.
2. **Shift creative budget toward Video and Stories**, which convert better per impression than Image or Carousel formats.
3. **Tailor messaging by market tier** — high-volume markets (India, Brazil-type audiences) vs. higher-value markets (Germany, UK, US) likely need different creative and offer strategies rather than one generic campaign.
4. **Fix the two data quality issues** (corrupted IDs, filter scope) before using this as a live reporting tool — both are quick fixes but currently limit trust in the exact totals.

## 9. Tools Used

- **Power BI Desktop** — data modeling, DAX, report design
- **Power Query (M)** — data loading and type transformation
- **DAX** — 11 custom measures including dynamic-title measures for the metric-switcher
- **Excel** — initial data inspection

## 10. Repository Structure

```
meta-ads-performance-dashboard/
├── README.md
├── dashboard/
│   └── Meta_Ads_Performance_Dashboard.pbix
├── data/
│   ├── ad_events.csv
│   ├── ads.csv
│   ├── campaigns.csv
│   └── users.csv
├── screenshots/
│   └── 01-overview.png
└── docs/
    └── kpi-definitions.md
```

## 11. How to View

1. Download `Meta_Ads_Performance_Dashboard.pbix` from the `dashboard/` folder.
2. Open in [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free).
3. The four CSVs in `data/` are the original source files, included for transparency and reproducibility.

---

**Author:** Siddhi Dubey — [LinkedIn](https://linkedin.com/in/siddhi-dubey242)
