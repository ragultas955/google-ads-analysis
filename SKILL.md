---
name: google-ads-monthly-report
description: Build a client-ready 8-slide monthly Google Ads performance dashboard (PPTX) comparing the reporting month with the previous month - overall KPIs, campaigns, search terms (bar charts and top 20), age groups (pie charts) and locations (bar charts), each with key insights. Use whenever the user asks for a Google Ads monthly report, Google Ads performance review, month-over-month Google Ads comparison, search term, age or location analysis for Google Ads, or a Google Ads PPT or dashboard, even if they do not say "skill" or "template". Uses a bundled Python script so the output is identical in Claude, Claude Code, ChatGPT, Codex and Gemini. Client, months, brand terms and data source are runtime inputs.
---

# Google Ads Monthly Performance Report

Produces the same 8-slide, dashboard-style PPTX on any agent. **The agent never designs the deck by hand.** The agent's job is to (1) collect and normalize data into CSVs, (2) run `scripts/build_report.py`, (3) check the result. Layout, charts, colours, numbers and default insight wording all come from the script, which is what makes results identical across ChatGPT, Codex, Claude, Claude Code and Gemini.

## Inputs (ask only for what is missing)

1. Client / brand name (never hardcode a client)
2. Reporting month and comparison month (default: the month before), e.g. "August 2026" and "July 2026"
3. Data source: uploaded Google Ads exports (CSV/XLSX) or a connected Google Ads account/MCP. Never invent or estimate numbers.
4. Brand terms to exclude from search term slides (brand name, spelling variants). If not given, ask once.
5. Competitor terms (optional, only used to tag the Type column; competitor terms are always kept)
6. Currency prefix (default: whatever the export uses)

## Workflow (follow in order)

1. **Install dependency** if missing: `pip install python-pptx`.
2. **Collect data for both months** and write normalized CSVs into a working folder. Exact columns are in `references/data-schema.md`. Every file has a `period` column with the value `current` or `previous`.
   - `account.csv` (account totals), `campaigns.csv`, `search_terms.csv`, `age.csv`, `location.csv`, optional `location_map.csv`.
3. **Resolve locations** to place names (e.g. Chennai, Chengalpattu). Never leave pincodes: put every pincode in `location_map.csv` (`raw,place`) using your best knowledge, and flag uncertain ones to the user. Unmapped pincodes fall into "Other" with a warning.
4. **Run the builder**:
   ```
   python scripts/build_report.py --data-dir <folder> --client "<Client>" \
     --month "August 2026" --prev-month "July 2026" \
     --brand-terms "brand,brand variant" --competitors "rival a,rival b" \
     --currency "Rs " --out "<Client>_Google_Ads_<Month>.pptx"
   ```
   The script excludes brand terms, keeps competitors, filters campaigns to those that ran in the reporting month, computes all metrics and month-over-month deltas, and builds all 8 slides.
5. **Read the script's WARNING lines** and fix or disclose them (unmapped pincodes, totals that do not reconcile, missing files).
6. **Visual check** if a renderer exists (e.g. `soffice --headless --convert-to pdf`, then rasterize): confirm no overflow or clipping. Do not restyle; if something is broken, fix the data or report the issue.
7. **Deliver** the PPTX plus a 3-line summary of biggest wins and concerns, and list assumptions (brand terms excluded, locations grouped, campaigns capped).

## Optional: custom insight wording

Default insights are rule-based and deterministic. If the user wants richer commentary, write an `insights.json` and pass `--insights insights.json`. Any key may be omitted:

```json
{
  "headline": "One-line summary for slide 1",
  "campaigns": {"<campaign name>": "short insight"},
  "search_terms": {"<term>": "short insight"},
  "age": {"25-34": "short insight"},
  "location": {"Chennai": "short insight"},
  "bullets": {"search_terms": ["b1","b2","b3","b4"], "age": ["..."], "location": ["..."]}
}
```
Rules for custom insights: each must cite a number from the data; exactly 4 bullets per Key Insights panel; be honest when a metric worsened; table insights stay under about 45 characters. Never change numbers, layout or slide order.

## Slide structure (fixed, built by the script)

1. Overall Performance: KPI cards (Impressions, Clicks, Conversions, Cost, Avg. CPC, CTR) with previous-month value and % change, plus a headline.
2. Campaign Performance: table of campaigns that ran in the reporting month, with Insight column and totals row.
3. Search Term Analysis: bar chart terms vs impressions and clicks, bar chart terms vs conversions (current vs previous), 4 Key Insights. Brand terms excluded, competitors kept.
4. Top 20 Search Terms: ranked table with MoM change and Insight column (Competitor/Generic tag when competitors supplied).
5. Age Group Analysis: pie of clicks, pie of conversions, 4 Key Insights.
6. Age Group Table with Insight column.
7. Location Analysis: bar chart location vs impressions and clicks, bar chart location vs conversions (current vs previous), 4 Key Insights. Place names only.
8. Location Performance Table (top 15 plus Others) with Insight column. Place names only.

## Metric rules (implemented in the script)

- CTR = clicks / impressions; Avg. CPC = cost / clicks; Cost per conversion = cost / conversions. Always recomputed from totals, never averaged.
- MoM change = (current - previous) / previous. For cost, CPC and cost per conversion a decrease is green (improvement). Previous = 0 shows "New".
- Top terms ranked by clicks, tie-break conversions then impressions. Age "Undetermined" is dropped from pies if under 5% of clicks.
- Impression-vs-click charts use a log scale (labelled) because impressions are far larger than clicks.

## Guardrails

- No client names, account IDs or real numbers are baked into this skill; `assets/sample_data/` is synthetic.
- Do not hand-build slides, change the theme or reorder slides; that breaks cross-platform consistency.
- Do not fabricate data. If a report is missing, the script leaves that slide with a "not provided" note; tell the user.
- Be honest in insights: if performance declined in an area, say so and recommend an action.
