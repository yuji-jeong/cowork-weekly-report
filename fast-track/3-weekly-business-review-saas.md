---
name: weekly-business-review-saas
description: "Generates a weekly business review (HTML artifact) for a SaaS product exec covering bookings and revenue vs. annual target, YTD pacing, pipeline coverage, net new pipeline, forecast category rollup, and portfolio-level conversion trend, deal size, and sales cycle from HubSpot (or a connected finance/billing tool where specified). Shareable as-is with the leadership team."
version: 1.0
author: yuji-jeong
data_sources:
  - HubSpot
  - "[Finance/billing tool, e.g. Stripe or Chargebee — optional, only add if connected and the user has named it as the source for a specific metric]"
trigger_phrases:
  - "run my weekly business review"
  - "how's the business tracking this week"
  - "pull the WBR"
  - "are we going to hit the number this year"
output_format: HTML artifact
---
# Weekly Business Review: SaaS
## Goal
This report answers one question at the start of each week: are we going to hit the annual number, and is anything happening right now that changes that. It is a **company-wide directional tool**, not a statistically validated forecast; small sample sizes at the portfolio level are still possible near quarter boundaries and that's fine. It is written to be **shareable with the leadership team or board as-is**: it must show what's going well, not just what's at risk. Target: a 60-second read that ends with the exec knowing whether the org is on pace and what, if anything, needs their attention this week.
---
## Prerequisites
- **HubSpot connector connected**: this is the default source for every metric in this report. Deal records, pipeline data, and the deal forecast category field all come from here unless a specific metric below names a different connected source.
- **Finance/billing tool connector (optional)**: only used for a given metric if the user has explicitly named a connected finance or billing tool (e.g., Stripe, Chargebee) as that metric's source of truth, most likely for recognized revenue. If no such tool is named or connected, revenue is read from HubSpot the same way bookings are, and the report notes this explicitly rather than silently treating them as the same number.
- **Annual bookings/revenue target in HubSpot Goals (company-wide, not per-rep)**: the report cannot generate without this number. If it cannot be found, Claude must ask the exec for it before proceeding. Do not generate the report without it.
- **Quarterly target**: used for the pipeline coverage ratio. If a company-wide quarterly target is not separately set in HubSpot Goals, derive it as the annual target ÷ 4 and state that assumption in a subline on the Pipeline Coverage card. Do not silently substitute this without disclosing it.
- **Currency**: read from the currency field on HubSpot deal records. If deal records show a single consistent currency, use it. If deals are mixed-currency or the currency cannot be determined, ask the exec which currency to report in before generating. Do not guess.
---
## Cadence Rule (which window each metric lives in)
Every metric in this report belongs to exactly one cadence. The cadence controls the time window, the delta treatment, and the section eyebrow.
- **Weekly metrics** (current 7-day window, WoW delta per the Global Delta Rule): bookings and revenue this week, net new pipeline created vs. lost, forecast category rollup, and the at-risk/win flags.
- **YTD metrics** (year-to-date cumulative window, WoW delta shown on the incremental weekly movement of the cumulative total, attainment % shown against the annual target, never a separate "YTD vs. YTD" delta): total bookings and revenue YTD vs. annual target, YTD pacing.
- **Quarterly metrics** (QTD window benchmarked against the quarterly target, **no WoW delta**): pipeline coverage ratio, QTD net new pipeline rollup.
- **Monthly/Quarterly metrics** (trailing window, **never a WoW delta**): portfolio conversion trend, average deal size, average sales cycle length. Week-over-week movement on these is noise from small weekly sample sizes, not signal; the exec acts on the month and the quarter. No ↑/↓ arrows appear anywhere on these metrics, and their eyebrows carry the window instead (e.g. "PORTFOLIO HEALTH · TRAILING QUARTER").
- Never mix cadences inside one visual element. A weekly element carries WoW deltas; a YTD, quarterly, or monthly element does not.
---
## North Star Metrics (one per decision)
| # | Decision | North Star Metric | Cadence |
|---|----------|-------------------|---------|
| 1 | Are we going to hit the number this year? | Total bookings and total revenue, YTD, each vs. the annual target | YTD |
| 2 | Are we pacing on track to get there? | YTD pacing: required $/week to hit target vs. actual weekly pace, plus projected year-end at current pace | YTD |
| 3 | Do we have enough pipeline to hit the number? | Pipeline coverage ratio: open pipeline ÷ remaining quarterly target | Quarterly |
| 4 | Is pipeline growing or shrinking? | Net new pipeline created vs. lost this week, plus the QTD rollup | Weekly + Quarterly |
| 5 | How much of the pipeline does the org actually believe will close? | Forecast category rollup: Commit / Best Case / Pipeline, each in $ | Weekly |
| 6 | Is the underlying sales motion healthy? | Portfolio-wide conversion rate, average deal size, average sales cycle length | Monthly/Quarterly |
| 7 | What needs my attention right now? | Largest at-risk deal ($ impact) and biggest win, this week | Weekly (point-in-time) |
| — | Did we move up or down vs. the prior period? | PoP delta on every metric above, per its own cadence per the Cadence Rule (inline on the metric itself, not a separate section) | — |
---
## Bookings vs. Revenue: Two Numbers, Never Blended
Bookings (closed-won deal value from HubSpot) and revenue (recognized revenue, from a finance/billing tool if one is connected and named for this purpose, otherwise the same closed-won value as bookings) are tracked as **two separate lines inside the same North Star 1 card**, each with its own WoW delta and its own attainment % against the annual target. They are never summed, averaged, or rendered as a single blended number, even when they happen to be numerically identical because no separate finance tool is connected. If they are numerically identical for this reason, state that plainly as a one-line note under the card rather than silently showing one number twice.
---
## Global Delta Rule (applies to every WoW number in the report)
This is the single, universal format for week-over-week change on Weekly-cadence metrics, and for the incremental weekly movement shown on YTD-cadence metrics. It applies to every metric, card, chart, and table cell it touches. No exceptions. Quarterly and Monthly/Quarterly-cadence metrics never render a delta slot at all, per the Cadence Rule.
- A WoW delta is always and only rendered as an arrow followed by the percentage to one decimal place: ↑16.7% or ↓4.2%.
- Never "prior week was X". Never a bare percentage without an arrow. Never an arrow without a percentage. Never "flat" or "=" in any delta slot; when a delta cannot be computed, the hyphen rule below applies.
- If the delta rounds to 0.0%, render ↑0.0% (zero or positive gets ↑, negative gets ↓).
- If a WoW delta cannot be computed because there is no prior data to compare, render a plain hyphen "-" in the delta slot. The hyphen means "no previous data to compare". Never "N/A" in a delta slot; "N/A" stays the indicator for missing VALUES in cells and cards.
- No suffix text on the delta itself (no "vs prior week", no "WoW"). Section eyebrows already carry the period context.
- ↑ and ↓ are reserved strictly for WoW deltas. A percentage that is not a WoW delta (attainment %, coverage ratio, conversion rate, forecast category share) never gets an arrow.
- Delta color: #c4c8d0 neutral by default. Green/amber/red only on the metrics listed in the Judgment Rules table, and even then the format stays arrow + one-decimal percentage.
---
## Judgment Rules: When Color Means an Opinion, and When It's Just a Fact
**The default: direction is a fact, not an opinion.** Whether a number went up or down is objectively true. Whether "up" is *good* usually depends on context the report doesn't have. So by default, direction is neutral.
- **Neutral metrics (no benchmark exists):** delta rendered per the Global Delta Rule in #c4c8d0 next to the number (Weekly/YTD cadence), or the plain value with no delta (Quarterly/Monthly cadence). No green, no red. Applies to: bookings and revenue $ values, net new pipeline created/lost, QTD net new pipeline rollup, forecast category values (Commit/Best Case/Pipeline), average deal size, average sales cycle length, biggest win, largest at-risk deal.
- **Judged metrics (a real benchmark exists):** the only places allowed to render an opinion, because there's a defensible number to compare against.
| Metric | Benchmark | Green (good) | Amber (monitor) | Red (bad) |
|---|---|---|---|---|
| YTD pacing (actual weekly bookings pace) | Required $/week to hit annual target | Actual weekly pace ≥ required pace | 80-99% of required pace | Below 80% of required pace |
| Pipeline coverage ratio | 3.0x remaining quarterly target (standard rule of thumb; some orgs run 4-5x depending on cycle length and close rate) | At or above 3.0x | 2.0x-2.9x | Below 2.0x |
Portfolio conversion rate is intentionally **not** a judged metric in this report: at the portfolio level with a monthly/quarterly cadence, a single trailing-window comparison is informative but not benchmarked against a hard external threshold the way pacing and coverage are. If a benchmark can't be computed (see Fallback Rules), fall back to a neutral #c4c8d0 rendering. Never guess a color.
---
## Accessory Metrics
### North Star 1 and 2: Revenue, Bookings, and Pacing
- **Bookings this week (closed-won deal value)**: neutral WoW delta, feeds the YTD bookings total.
- **Revenue this week**: neutral WoW delta, feeds the YTD revenue total; same source as bookings unless a finance/billing tool is connected and named.
- **Weeks elapsed / weeks remaining in the year**: used to compute required weekly pace and projected year-end at pace.
- **Required $/week to hit annual target**: (Annual target - YTD bookings) ÷ weeks remaining in year. Benchmark used to judge YTD pacing.
- **Projected year-end at pace**: (YTD bookings ÷ weeks elapsed) × 52. Neutral extrapolation, not a judgment; its delta slot always renders a plain hyphen "-" because it is a projection with no prior-week comparison, never "N/A".
### North Star 3: Pipeline Coverage
- **Total open pipeline value**: feeds the numerator. No standalone delta shown; it appears only inside the ratio math subline.
- **Remaining quarterly target**: quarterly target - QTD closed-won bookings. Feeds the denominator.
### North Star 4: Net New Pipeline
- **New pipeline created this week**: neutral WoW delta.
- **Pipeline lost this week**: value of deals marked Closed Lost or otherwise removed from open pipeline this week, neutral WoW delta.
- **Net new pipeline this week** (created minus lost): neutral WoW delta, the headline number on the card.
- **QTD net new pipeline rollup** (created minus lost, quarter to date): plain running total, no delta slot, per the Cadence Rule; shows whether this week's direction is a blip or a pattern.
### North Star 5: Forecast Category Rollup
- **Commit $ value**: neutral WoW delta.
- **Best Case $ value**: neutral WoW delta.
- **Pipeline (uncategorized/no forecast category) $ value**: neutral WoW delta.
- **Final-weeks-of-quarter callout**: if the report date falls within the final 2 weeks of the current quarter, render one additional observational line above the composite bar noting that Commit should be tightening toward the total forecast. This is a factual, non-judged callout, not a colored benchmark.
### North Star 6: Portfolio Health
- **Portfolio-wide conversion rate**: COUNT(advanced past a defined stage) ÷ COUNT(entered), trailing month or quarter (whichever the org's HubSpot data supports; default to trailing quarter if both are available). No delta, per the Cadence Rule.
- **Average deal size (closed-won, trailing period)**: no delta.
- **Average sales cycle length (closed deals, trailing period)**: no delta.
### North Star 7: Flags
- **Largest at-risk deal**: among open deals in the Commit or Best Case forecast category, flag by dollar materiality: the single largest deal (by $ value) that shows at least one risk signal (close date has passed, 14+ days without activity, or forecast category was downgraded this week). If multiple deals qualify, only the single largest by $ value renders; this is one flag, not a list. Include deal name, $ value, and the specific risk signal that triggered the flag.
- **Biggest win**: the single largest Closed Won deal this week by $ value. Include deal name, $ value, and account/company name for context.
---
## Visualization Guide
| Metric | Message type | Chart to use | Why this chart |
|--------|-------------|--------------|----------------|
| Bookings & Revenue YTD + Pacing | Part-of-whole (progress) | One composite card: horizontal progress bar (YTD attainment % vs. 100%) with bookings and revenue as two separate labeled lines above it, each with its own WoW delta; pacing sub-KPI row below (required $/week in judged color, projected year-end neutral) | Single dominant card matches the report's biggest decision; two labeled lines keep bookings and revenue legible without a second chart |
| Pipeline coverage ratio | Single number, judged | KPI card, "<X.X>x" in judged color, benchmark stated inline, math subline showing open pipeline and remaining target | A ratio against an explicit benchmark; color is earned because 3:1 is a real reference point |
| Net new pipeline (created vs. lost) | Comparison + WoW | One bar per category (created, lost), Created fill #3b82f6 and Lost fill #ef4444, net new headline number with its WoW delta above the bars, QTD rollup as a plain value line below with no delta | Two categories only; a bar-per-category plus a headline net number reads faster than a stacked or grouped chart |
| Forecast category rollup | Part-of-whole | One horizontal composite bar with three segments (Commit, Best Case, Pipeline), each segment labeled with its $ value and WoW delta printed on the segment itself | Three categories, ordered by confidence; a single segmented bar shows the mix at a glance without a legend |
| Portfolio health (conversion, deal size, cycle length) | Comparison | One 3-up KPI row, no deltas per the Cadence Rule | Three scalars on the same trailing window; a KPI row is faster to scan than three separate charts |
| Flags: at-risk deal + biggest win | Operational | Small card, two labeled lines, no chart | Two named items with dollar amounts; a chart adds no value to a two-item flag |
| WoW delta | Modifier | Inline ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0 by default, Weekly/YTD-cadence elements only | The delta annotates the number it belongs to; never its own chart or section |
**Chart rules:**
- **No duplicate bar groups for "this week vs. prior week."** Use one bar per category plus an inline delta per the Global Delta Rule.
- **Merge charts that share an axis** before adding a new one; do not split bookings and revenue into separate cards, and do not split pacing out of the Revenue & Bookings card.
- **Any derived or secondary number must be attached directly to the element it describes**: printed on or inside the bar or segment it belongs to, never floated as separate text nearby (this is why each forecast category segment carries its own $ value and delta).
- Every chart title states the finding, not the variable names.
- Color is either: (a) #c4c8d0 delta-only, or (b) green/amber/red per the Judgment Rules table. Never green/red on a metric without a listed benchmark (the Lost pipeline bar fill and the at-risk flag border are fixed categorical uses, not judgments).
- Labels ≤ 3 words where possible.
- If a metric VALUE cannot be computed, show "N/A" in the card (a Weekly/YTD delta slot with no prior data shows a plain hyphen "-" per the Global Delta Rule). Never leave a card blank or skip it.
- **Cap total distinct visual elements at 6 across the report.** The ledger: (1) Revenue & Bookings YTD + Pacing composite card, (2) Pipeline Coverage card, (3) Net New Pipeline card, (4) Forecast Category Rollup composite bar, (5) Portfolio Health KPI row, (6) Flags card. Merge or cut before rendering if a draft exceeds this; do not add a seventh without removing one.
---
## Data Source Mapping
| Metric | Source | Field / Object | Notes |
|--------|--------|----------------|-------|
| Annual bookings/revenue target | HubSpot Goals | Goals API: company-wide target for current year | If not found, stop and ask the exec before generating |
| Quarterly target | HubSpot Goals | Goals API: company-wide target for current quarter | If not separately set, derive as annual target ÷ 4 and disclose the assumption on the Pipeline Coverage card |
| Bookings YTD | HubSpot (default) | SUM(deals.amount) WHERE stage = Closed Won AND close_date within current year | Bookings and revenue are tracked as two separate lines even if numerically identical |
| Revenue YTD | HubSpot, or the connected finance/billing tool if named | deals.amount (HubSpot) or the recognized-revenue field/report from the named tool | Note explicitly if this equals bookings because no separate finance source is connected |
| Bookings/revenue this week | Same as above | Filtered to current 7-day window | Feeds the WoW delta on each YTD line |
| Required $/week to hit annual target | Derived | (Annual target - YTD bookings) ÷ weeks remaining in year | Benchmark for judging YTD pacing |
| Projected year-end at pace | Derived | (YTD bookings ÷ weeks elapsed) × 52 | Neutral display; this is an extrapolation, not a judgment |
| Total open pipeline value | HubSpot | SUM(deals.amount) WHERE stage ≠ Closed Won AND stage ≠ Closed Lost | |
| Remaining quarterly target | Derived | Quarterly target - QTD closed-won bookings | Denominator for pipeline coverage ratio |
| Pipeline coverage ratio | Derived | Total open pipeline value ÷ remaining quarterly target | Judged vs. 3.0x |
| New pipeline created this week | HubSpot | deals.create_date within current 7-day window | |
| Pipeline lost this week | HubSpot | SUM(deals.amount) WHERE stage = Closed Lost OR removed from open pipeline, within current 7-day window | |
| Net new pipeline (weekly and QTD) | Derived | Pipeline created - pipeline lost, over the 7-day window (weekly) and quarter to date (QTD rollup) | QTD rollup has no delta slot per the Cadence Rule |
| Forecast category rollup | HubSpot | SUM(deals.amount) GROUP BY deal forecast category (Commit / Best Case / Pipeline), current + prior 7-day snapshot for the WoW delta | Deals with no forecast category set are excluded from this rollup and noted if the excluded value is material |
| Portfolio conversion rate | HubSpot | COUNT(advanced past defined stage) ÷ COUNT(entered), trailing month or quarter | Portfolio-wide, not per rep; no WoW delta |
| Average deal size (closed-won) | HubSpot | AVG(deals.amount), Closed Won, trailing month or quarter | No delta |
| Average sales cycle length | HubSpot | AVG(close_date - create_date) for Closed Won + Closed Lost, trailing month or quarter | No delta |
| Largest at-risk deal | HubSpot | Largest deals.amount WHERE forecast_category IN (Commit, Best Case) AND (close_date < today OR last_activity_date < today - 14 OR forecast_category downgraded this week) | Single largest by $ value only, not a list |
| Biggest win | HubSpot | Largest deals.amount WHERE stage = Closed Won AND close_date within current 7-day window | Include company/account name |
| WoW delta (Weekly/YTD metrics) | Derived | (current 7-day value - prior 7-day value) ÷ prior 7-day value × 100 | Rendered per the Global Delta Rule; color per Judgment Rules; Quarterly/Monthly metrics never compute this |
---
## Report Structure
Render the header block first and the footer last (both specified in the Style Contract). Produce sections in this exact order:
1. **Summary: What's Going On** *(mandatory, always first, bulleted, not prose)*
   Two bullet groups:
   - **"Wins This Week"**: 2-4 bullets. Achievements and positive movement vs. last week, with a number in each (e.g. "Bookings hit $412K this week vs. $336K last week (↑22.6%)"). This goes first; the report can't read as only a list of problems.
   - **"Focus For This Week"**: 2-4 bullets. Specific, actionable gaps naming the actual deal or category (e.g. "Pipeline coverage sits at 2.1x, below the 3.0x benchmark" or "Meridian Corp at $180K is the largest at-risk deal, no activity in 19 days"). Bullet counts don't need to be even; use however many genuinely apply.
   Each bullet ≤20 words and carries a number. No "significant," "up," or "trending" without a figure. Prior-week numbers may appear in Summary prose for context ("vs. $336K last week"), but the change itself always renders per the Global Delta Rule in parentheses; never a bare "up 22.6%" without the arrow. If there's genuinely no win to report, state a neutral fact instead of fabricating one (e.g., "Bookings held flat week over week").
2. **Revenue & Bookings** *(YTD; dominant composite card: progress bar showing YTD attainment % vs. 100% target; bookings and revenue as two separately labeled lines above it, each with its own $ value and WoW delta; pacing sub-KPI row below with required $/week in judged color and projected year-end neutral with its "-" delta slot; full anatomy in the Style Contract)*
3. **Pipeline Coverage** *(Quarterly; single KPI card: ratio in judged color, benchmark stated inline, math subline showing open pipeline and remaining quarterly target; note if the quarterly target was derived from the annual target)*
4. **Net New Pipeline** *(Weekly; created vs. lost bars with the net new headline number and its WoW delta above them; QTD rollup as a plain value line below, no delta)*
5. **Forecast Category Rollup** *(Weekly; single composite bar with three labeled segments, Commit, Best Case, Pipeline, each carrying its own $ value and WoW delta; final-weeks-of-quarter callout line above the bar only when applicable)*
6. **Portfolio Health** *(Monthly/Quarterly; 3-up KPI row: conversion rate, average deal size, average sales cycle length, no deltas per the Cadence Rule)*
7. **Flags: What Needs Attention** *(Weekly, point-in-time; small card, two labeled lines: largest at-risk deal with its risk signal and $ value, biggest win with its $ value and account name)*
---
## Design Language
- **UI library:** Plain HTML + inline CSS (fully self-contained artifact, no external dependencies)
- **Visual style:** Bloomberg terminal × Metabase dark mode; data-dense, zero decoration
- **Punctuation, hard rule: no em dashes or en dashes anywhere in the rendered report.** Not in titles, subtitles, section headers, labels, captions, table cells, notes, bullets, or chips. Use colons, commas, periods, or the middle dot ( · ) separator instead. Plain hyphens are allowed; the ban is on em and en dashes. Date ranges use a plain hyphen (Jul 7-13). Missing VALUES render as "N/A"; a WoW delta slot with no prior data renders a plain hyphen "-" per the Global Delta Rule.
- **Number formatting:** dollar amounts ending in three zeros abbreviate with K ($280,000 renders as "$280K"); larger round amounts abbreviate with M ($4,200,000 renders as "$4.2M"); non-round amounts render in full. The currency code appears exactly once, in the footer (e.g. "All figures in USD"), determined per the Prerequisites currency rule, never appended to individual numbers.
- **Background:** #0f1117
- **Surface / card background:** #1a1d27
- **Primary text:** #f0f0f0
- **Secondary / label text:** #c4c8d0; must meet **WCAG AA contrast (≥4.5:1)** against whatever it sits on. Verify against both page background and card background before finalizing.
- **Text rendered inside a colored bar/segment** (e.g., the forecast category segments) must contrast against *that segment's fill color*, not the page background; typically near-white on saturated fills. Check each fill color individually.
- **Accent color:** #3b82f6 (blue): progress bars, active highlights, links, forecast category fills, created-pipeline bar fill
- **Judged-good:** #22c55e (green), only on metrics listed in the Judgment Rules table
- **Judged-monitor:** #f59e0b (amber), only on metrics listed in the Judgment Rules table
- **Judged-bad:** #ef4444 (red), only on metrics listed in the Judgment Rules table (fixed categorical uses excepted: the lost-pipeline bar fill and the at-risk flag border)
- **Neutral delta (everything else):** ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0, never green/red
- **At-risk deal flag:** #ef4444 left border on its line in the Flags card
- **Font:** one sans-serif stack for everything, including numbers: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif. Numbers are bold with font-variant-numeric: tabular-nums. No monospace, no webfonts, no external fonts.
- **Font size floor: 14px everywhere, no exceptions**, including chart annotations and badges. Nothing in the report may render smaller than 14px.
- **Number size:** 28px for North Star numbers; 18px for accessory metrics; 14px for labels and chart annotations
- **Layout:**
  - Full-width single column
  - KPI cards in a 3-up row max
  - Delta/judgment badge inline, right-aligned, on the metric itself
- **One dominant focal point:** the Revenue & Bookings YTD progress bar card is the largest element on the page.
- **Chart titles state the finding**, not the variable names.
- **6-visual-element cap** is a hard constraint, not a suggestion.
- **Before shipping, spot-check every text/background pairing used in the report against a contrast checker.** This is a required step, not implied by picking colors from this palette.
---
## Style Contract (reproduce this exact rendering every run)
The report must look identical from run to run, and identical in style to the other report skills in this system. Do not restyle, reinterpret, or "improve" the visual design. Implement exactly:
- **Page:** background #0f1117, content max-width 1100px, 32px page padding, single column, 16px vertical gap between cards.
- **Section eyebrows:** every section opens with an eyebrow line sitting above its card, outside it: 14px, uppercase, letter-spacing 0.08em, #c4c8d0, middle-dot separators. Weekly sections carry the week (e.g. "NET NEW PIPELINE · WEEK OF JUL 7-13"); YTD sections carry the year (e.g. "REVENUE & BOOKINGS · YTD 2026"); Quarterly and Monthly/Quarterly sections carry their own period (e.g. "PIPELINE COVERAGE · Q3 2026", "PORTFOLIO HEALTH · TRAILING QUARTER").
- **Header block:** eyebrow "WEEKLY BUSINESS REVIEW · <quarter>"; company/org name 24px bold #f0f0f0; subline 14px #c4c8d0 "Week of <Jul 7-13, 2026> · Generated <date>". Top-right status chip: #1a1d27 pill, 1px border rgba(255,255,255,0.08), border-radius 999px, 8px colored status dot (green/amber/red per the pipeline coverage judgment) + 14px text (e.g. "Coverage 3.2x, above benchmark").
- **Cards:** #1a1d27 background, 1px solid rgba(255,255,255,0.07) border, border-radius 10px, 20-24px padding. Card-internal section titles: 14px uppercase letterspaced #c4c8d0.
- **Summary card:** 3px #3b82f6 left border; two-column layout; group headers 14px uppercase: "✓ WINS THIS WEEK" in #22c55e, "⚑ FOCUS FOR THIS WEEK" in #f59e0b; bullets 14px #f0f0f0 with dim bullet glyphs.
- **KPI pattern:** label 14px #c4c8d0 above the number; number 28px (North Star) or 18px (accessory) bold #f0f0f0; WoW delta inline immediately after the number, 14px, per the Global Delta Rule, Weekly/YTD-cadence cards only.
- **Revenue & Bookings card anatomy:** internal title "YTD PROGRESS: <TARGET> ANNUAL TARGET" with the target abbreviated per the number formatting rule. Two labeled lines above the progress bar: "Bookings" and "Revenue", each 28px bold #f0f0f0 with its WoW delta inline; if numerically identical because no separate finance source is connected, a 14px #c4c8d0 note directly beneath both: "Revenue equals bookings: no separate finance/billing source connected." Progress bar and axis below (0 / "Week X of 52" / target). Then a 2-up sub-KPI row: (1) Required $/Week to Hit Target in the judged color, meaning the dollar amount text itself renders in that color, with a green check mark immediately to its right when judged green, and directly below it a line in the same judged color stating actual pace vs. target (e.g. "Actual $96K/week · 12% above target" or "below target"); (2) Projected Year-End at Pace with subline "<X>% of target · neutral estimate", its delta slot always a plain hyphen "-", never "N/A".
- **Progress bar:** 8px tall, border-radius 4px, fill #3b82f6, track #262a36; axis line below: left "0", center "Week X of 52", right the target amount abbreviated per the number formatting rule, all 14px #c4c8d0.
- **Pipeline Coverage card anatomy:** internal title "COVERAGE VS. REMAINING QUARTERLY TARGET"; dominant number "<X.X>x" 28px bold in the judged color; immediately right of it, 14px #c4c8d0 "benchmark 3.0x"; directly below, one 14px #c4c8d0 subline: "<open pipeline> open pipeline · <remaining> remaining quarterly target". If the quarterly target was derived from the annual target, add a second 14px #c4c8d0 subline: "Quarterly target derived as annual target ÷ 4; no separate quarterly target set in HubSpot Goals." No delta slot.
- **Net New Pipeline card anatomy:** headline "Net New Pipeline: <value>" 28px bold #f0f0f0 with its WoW delta inline. Below it, one bar per category (Created, Lost): Created fill #3b82f6, Lost fill #ef4444, category label left 14px #c4c8d0, value and WoW delta right 14px #c4c8d0. Below the bars, one plain 14px #c4c8d0 line: "QTD rollup: <value> net new pipeline" with no delta.
- **Forecast Category Rollup card anatomy:** if within the final 2 weeks of the current quarter, one 14px #f59e0b callout line above the bar: "Final 2 weeks of Q<X>: Commit should be tightening toward the total forecast." Composite bar: 24-28px tall, rounded corners, three segments left to right in order Commit, Best Case, Pipeline, fill #3b82f6 with progressively lighter tints across the three segments; each segment's category name, $ value, and WoW delta printed on the segment itself, text color contrasted against that segment's fill.
- **Portfolio Health row anatomy:** 3-up KPI row, no deltas: Conversion Rate, Average Deal Size, Average Sales Cycle Length, each 18px bold #f0f0f0 with its 14px #c4c8d0 label above and trailing-window note below (e.g. "trailing quarter").
- **Flags card anatomy:** two labeled lines. "Largest At-Risk Deal": deal name, $ value, and risk signal (e.g. "no activity in 19 days"), 3px #ef4444 left border on this line. "Biggest Win": deal name, $ value, and account/company name, no colored border (a win is a fact, not a judged metric). If no deal qualifies as at-risk this week, render "No deals currently flagged at risk." in place of that line. If no win closed this week, render "No deals closed this week." in place of that line.
- **Pills:** category pills (forecast category, risk signal type) where used in table or card context: #262a36 background, 4px radius, 14px text.
- **Footer:** hairline top border; left side "<Company/org name> · Week of <range> · <Quarter> · All figures in <currency>"; right side "Data: HubSpot<· finance/billing tool if connected> · weekly-business-review-saas v1.0"; both 14px #c4c8d0.
- If a styling question is not answered here, resolve it with the nearest token above. Never introduce a new color, font, size, or component style.
---
## Output Contract
- **File type:** Single self-contained HTML file (all CSS inline, no external scripts or fonts)
- **Delivery method:** Rendered as an artifact in the chat
- **Required sections checklist:**
  - [ ] Header block + status chip (per Style Contract, chip judged on pipeline coverage ratio)
  - [ ] Summary / What's Going On (mandatory, always first, bulleted: Wins This Week + Focus For This Week)
  - [ ] Revenue & Bookings (bookings and revenue as two separate labeled lines, never blended; pacing sub-KPI row)
  - [ ] Pipeline Coverage (judged vs. 3.0x, benchmark stated inline, quarterly-target-derivation note if applicable)
  - [ ] Net New Pipeline (created vs. lost bars + QTD rollup line)
  - [ ] Forecast Category Rollup (three labeled segments, final-weeks-of-quarter callout when applicable)
  - [ ] Portfolio Health (3-up KPI row, no deltas)
  - [ ] Flags: What Needs Attention (at-risk deal + biggest win, single largest each, not a list)
  - [ ] Footer (per Style Contract)
- **Tone:** Operational and board/exec-shareable; specific numbers and named deals, achievements included, no fluff. Written for the exec ("your business"/"the org"), not first-person rep voice.
- **Punctuation:** zero em dashes and zero en dashes anywhere in the output, including titles, labels, and table cells. Missing values are "N/A"; a delta slot with no prior data to compare is a plain hyphen "-".
- **Reading level and length:** Summary is bulleted, not prose; 4-8 bullets total, ≤20 words each. Cards are not subject to this limit, but no text anywhere renders below the 14px floor.
- **Total visual elements:** ≤6 across the whole report, per the ledger in the Visualization Guide.
- **WoW display:** inline on every Weekly/YTD-cadence metric, formatted per the Global Delta Rule (arrow + one-decimal %), neutral #c4c8d0 by default. Quarterly and Monthly/Quarterly-cadence metrics (pipeline coverage, QTD net new pipeline rollup, portfolio health) never render a WoW delta or an arrow of any kind, per the Cadence Rule.
- **Style:** must match the Style Contract exactly, run to run.
---
## Fallback Rules
| Scenario | Claude should… |
|----------|----------------|
| Annual target not found in HubSpot Goals | Stop. Ask the exec for the annual target. Do not generate any part of the report until provided. |
| Quarterly target not separately set in HubSpot Goals | Derive it as annual target ÷ 4 and state that assumption in the Pipeline Coverage card subline. Do not block. |
| Currency mixed or cannot be determined from HubSpot deal records | Stop. Ask the exec which currency to report in before generating. |
| No finance/billing tool connected or named for revenue | Read revenue from HubSpot the same way as bookings, and add the "Revenue equals bookings" note per the Style Contract. Do not block. |
| No Closed Won deals this week | Show bookings/revenue this week as $0 with a neutral delta. Do not skip the section. |
| Prior 7-day window returns no data | Show a plain hyphen "-" in every affected delta slot (it means "no previous data to compare") plus a one-line note: "No prior week data available." Missing VALUES in cells still render "N/A". |
| No genuine "win" to report this week | State a neutral factual bullet in the Summary instead of fabricating a positive spin (e.g., "Bookings held flat week over week"). |
| No deal qualifies as at-risk this week | Render "No deals currently flagged at risk." in the Flags card. Do not skip the card. |
| No deal closed this week | Render "No deals closed this week." in the Flags card in place of the biggest win line. |
| Forecast category field missing on some deals | Exclude those deals from the rollup; if the excluded value is material (more than 10% of total open pipeline), add a one-line note under the composite bar stating the excluded amount. |
| Portfolio conversion rate cannot be calculated | Show "N/A" with note: "Insufficient closed deal volume in the trailing window." |
| Average sales cycle length cannot be calculated | Show "N/A" in that KPI. |
| A metric field is missing on a deal record | Show "N/A" in that cell. Never infer or fill in missing values. |
| HubSpot (or a connected finance/billing tool) returns an error on any query | Note the error at the top of the affected section. Proceed with all other sections. |
---
## What This Skill Is NOT For
- Do not use em dashes or en dashes anywhere in the rendered report: no titles, labels, captions, cells, or notes.
- Do not express a WoW delta in any format other than the Global Delta Rule (arrow + one-decimal %). No "prior week was X", no bare percentages, no arrows without percentages.
- Do not render a WoW delta, or any ↑/↓ arrow, on a Quarterly or Monthly/Quarterly-cadence metric (pipeline coverage, QTD net new pipeline rollup, portfolio health). Cadence is a hard boundary.
- Do not put ↑/↓ arrows on percentages that are not WoW deltas (e.g., pipeline coverage ratio, YTD attainment %, conversion rate).
- Do not blend bookings and revenue into a single number, even when they are numerically identical because no separate finance source is connected; state that explicitly instead.
- Do not render per-rep or per-team breakdowns anywhere in this report; that is the team-weekly-report skill's job, not this one. This report is company-wide/portfolio-level only.
- Do not include more than one at-risk deal or one win in the Flags card; it is a materiality-based single flag each, not a running list.
- Do not show internal deal IDs anywhere in the report; always use deal and account names.
- Do not color-code direction as good/bad on any metric without a listed benchmark. Portfolio conversion rate is intentionally neutral even though it is a health signal.
- Do not render two bar groups for "this week vs. prior week"; use the single-element-plus-inline-delta pattern.
- Do not float a derived rate (like a forecast category share) as separate text near a chart; attach it to the segment or bar it describes.
- Do not write the Summary as a paragraph; it must be two bulleted groups, wins first.
- Do not present the report as only a list of problems; it must show what's working before what's at risk.
- Do not fabricate a "win" bullet if there isn't a real one; state a neutral fact instead.
- Do not render any text below the 14px floor, including chart annotations.
- Do not generate forecast models or predict close probability; use HubSpot's own forecast category field.
- Do not create new CRM records, update deal stages, or log activities.
- Do not compare against periods other than those defined by the Cadence Rule unless explicitly asked.
- Do not fabricate missing fields; surface the gap instead.
- Do not exceed 6 total visual elements; if content doesn't fit, cut or merge, don't shrink the font.
- Do not deviate from the Style Contract: no new colors, fonts, sizes, or component styles.
---
## Example Trigger
> "Can you pull the weekly business review? Board update is Friday and I want to know if we're on pace to hit the year."
> "Run my weekly business review."
> "How's the business tracking this week?"
> "Are we going to hit the number this year?"
