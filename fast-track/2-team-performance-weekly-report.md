---
name: team-weekly-report
description: "Generates a weekly team performance report (HTML artifact) for a sales manager covering team pipeline by stage, per-rep pipeline, pipeline movement, a rep leaderboard, at-risk deals, per-rep quota pace (MTD/QTD), coverage ratio, and sales cycle / time-in-stage from HubSpot. Built for a high-velocity/SMB sales motion."
version: 1.0
author: yuji-jeong
data_sources:
  - HubSpot
trigger_phrases:
  - "run my team report"
  - "how's the team pipeline looking"
  - "which reps are behind"
  - "pull the team weekly report"
output_format: HTML artifact
---
# Weekly Team Performance Report
## Goal
This report answers one question at the start of each week: is the team moving in the right direction, which reps need my attention, and which deals are about to fall through the cracks? It is a **directional management tool**, not a statistically validated forecast. Small sample sizes are expected and fine. Target: a 90-second read that ends with the manager knowing exactly which rep and which deals to talk about this week.

This report is built for a **high-velocity/SMB sales motion**, where activity volume (calls and emails logged) is a first-class management signal. In a complex enterprise motion, activity counts matter less than stage progression; this skill does not attempt to serve that motion.
---
## Prerequisites
- **HubSpot connector connected**: deal records, activity logs, pipeline data, owner assignments, and quota targets all come from here.
- **Per-rep quota targets in HubSpot Goals**: the quota pace section needs a quota for each rep on the team. If a rep's quota cannot be found, render "N/A" in that rep's pace row with a note; do not block the report.
- **Team quota**: SUM of the per-rep quotas found in HubSpot Goals. If any rep's quota is missing, compute the team quota from the reps that have one and note the gap. The coverage ratio cannot render without a team quota; if none of the rep quotas can be found, ask the manager for the team number before generating.
---
## Cadence Rule (which window each metric lives in)
Every metric in this report belongs to exactly one cadence. The cadence controls the time window, the delta treatment, and the section eyebrow.
- **Weekly metrics** (current 7-day window, WoW delta per the Global Delta Rule): pipeline value by stage (team-wide and per-rep), pipeline movement (new, stalled, won, lost), the rep leaderboard (win rate, activity volume, deals closed), and the at-risk deals flag.
- **Monthly/Quarterly metrics** (MTD and/or QTD windows, **never a WoW delta**): quota attainment vs. target, coverage ratio, and sales cycle / time-in-stage. Week-over-week movement on these numbers is noise, not signal; managers act on the month and the quarter. No ↑/↓ arrows appear anywhere on these metrics, and their eyebrows carry the window instead (e.g. "QUOTA PACE · MTD + QTD · Q3 2026").
- Never mix cadences inside one visual element. A weekly element carries WoW deltas; a monthly/quarterly element carries none.
---
## North Star Metrics (one per question)
| # | Decision | North Star Metric | Cadence |
|---|----------|-------------------|---------|
| 1 | Where does the team's pipeline sit? | Team-wide pipeline value by stage | Weekly |
| 2 | Who owns that pipeline? | Per-rep open pipeline value by stage | Weekly |
| 3 | What moved this week? | Pipeline movement: new, won, lost, newly stalled | Weekly |
| 4 | Who is performing and who needs help? | Per-rep leaderboard: deals closed, win rate, activity volume | Weekly |
| 5 | Which deals are about to fall through the cracks? | At-risk deals: team-wide stalled count + the rep-level list | Weekly |
| 6 | Which specific reps are behind on the number? | Per-rep quota attainment vs. target, MTD + QTD | Monthly/Quarterly |
| 7 | Are we going to hit the number? | Coverage ratio: open pipeline ÷ remaining team quota, vs. 3:1 | Monthly/Quarterly |
| 8 | Is our process slow, and where? | Average sales cycle length + time-in-stage, with the dragging stage/rep surfaced | Monthly/Quarterly |
---
## Rep-Level vs. Team-Level (who gets sliced, who gets aggregated)
- **Rep-level only** (broken out by name, never as a team average): the leaderboard (comparing individuals is the whole point) and quota attainment vs. target (the manager needs to know which specific reps are behind; a blended team attainment number hides exactly that and is never rendered).
- **Team-level only** (one aggregate number, never sliced per rep): the coverage ratio (pipeline dollars vs. team quota is territory math; eight individual coverage ratios are noise, not signal) and the team-wide stage funnel (the aggregate funnel shape is the point).
- **Both** (team headline, rep-level drill attached): sales cycle / time-in-stage (team average answers "is our process slow"; the dragging rep is named inline on the worst stage bar) and at-risk deals (the team-wide count is the headline, but it is useless without the rep-level list, since someone has to go act on each deal).
---
## Global Delta Rule (applies to every WoW number in the report)
This is the single, universal format for week-over-week change. It applies to every Weekly-cadence metric, card, chart, and table cell. No exceptions. Monthly/Quarterly-cadence metrics never render a delta slot at all, per the Cadence Rule.
- A WoW delta is always and only rendered as an arrow followed by the percentage to one decimal place: ↑16.7% or ↓4.2%.
- Never "prior week was X". Never a bare percentage without an arrow. Never an arrow without a percentage. Never "flat" or "=" in any delta slot; when a delta cannot be computed, the hyphen rule below applies.
- If the delta rounds to 0.0%, render ↑0.0% (zero or positive gets ↑, negative gets ↓).
- If a WoW delta cannot be computed because there is no prior data to compare, render a plain hyphen "-" in the delta slot. The hyphen means "no previous data to compare". Never "N/A" in a delta slot; "N/A" stays the indicator for missing VALUES in cells and cards.
- No suffix text on the delta itself (no "vs prior week", no "WoW"). Section eyebrows already carry the week context.
- ↑ and ↓ are reserved strictly for WoW deltas. A percentage that is not a WoW delta (stage conversion rate, attainment %, a win rate value itself, coverage ratio) never gets an arrow.
- Delta color: #c4c8d0 neutral by default. Green/amber/red only on the judged metrics in the Judgment Rules table, and even then the format stays arrow + one-decimal percentage.
---
## Judgment Rules: When Color Means an Opinion, and When It's Just a Fact
**The default: direction is a fact, not an opinion.** Whether a number went up or down is objectively true. Whether "up" is *good* usually depends on context the report doesn't have. So by default, direction is neutral.
- **Neutral metrics (no benchmark exists):** delta rendered per the Global Delta Rule in #c4c8d0 next to the number (Weekly cadence), or the plain value with no delta (Monthly/Quarterly cadence). No green, no red. Applies to: pipeline value by stage (team and per rep), new deals created, closed-won and closed-lost counts and values, calls logged, emails logged, deals closed per rep, average sales cycle length, time-in-stage averages, stalled deal count.
- **Judged metrics (a real benchmark exists):** the only places allowed to render an opinion, because there's a defensible number to compare against.
| Metric | Benchmark | Green (good) | Amber (monitor) | Red (bad) |
|---|---|---|---|---|
| Per-rep win rate (weekly) | That rep's own trailing 8-week win rate | At or above benchmark | Within 10 pts below | More than 10 pts below |
| Per-rep quota pace (MTD and QTD) | Expected attainment at this point in the period (days elapsed ÷ days in period) | At or above expected | 80-99% of expected | Below 80% of expected |
| Coverage ratio (QTD) | 3.0x remaining team quota | At or above 3.0x | 2.0x-2.9x | Below 2.0x |
The 3:1 coverage benchmark is the standard industry rule of thumb (some orgs run 4-5x depending on cycle length and close rate); render the benchmark explicitly in the card so the judgment is legible.
If a benchmark can't be computed (see Fallback Rules), fall back to a neutral #c4c8d0 rendering. Never guess a color.
---
## Accessory Metrics
### Section 2: Team Pipeline by Stage (Weekly)
- **Deals by stage + stage conversion rate, team-wide**: one composite pipeline visual, two stacked parts in the same card, aggregated across every rep on the team. Part A: the horizontal stage funnel, one bar per open stage, bar length proportional to SUM(amount), stage name left; on the right, the deal count with its unit, the value, and that stage's WoW value delta (e.g. "23 deals · $412.6K ↑4.1%"). Never a bare count: "23" alone is ambiguous, it must read "23 deals". Part B: the stage conversion column row rendered directly below it (exact layout in the Visualization Guide). Conversion is COUNT(advanced) ÷ COUNT(entered) per stage over a trailing 90-day window, a different time window than the rest of the report; the conversion row must say so via a "(90-day)" tag in its title plus the required caption below it. Conversion percentages live only in Part B, never as badges on the funnel bars.
- This composite visual is the **dominant focal point** of the report: managers live in this view, so it is the largest element on the page.
### Section 3: Pipeline by Rep (Weekly)
- **Per-rep open pipeline value by stage**: one horizontal stacked bar per rep, segments colored with the same stage tints as the team funnel, rep name left; on the right, "<N> deals · <value>" plus the WoW delta of that rep's total open value per the Global Delta Rule. Sorted by open value descending. No conversion percentages here; volume and its WoW delta only.
### Section 4: Pipeline Movement This Week (Weekly)
- **New deals created this week**: neutral delta only.
- **Closed won / closed lost this week**: one combined KPI card, both counts with their values, neutral deltas (won/lost counts have no benchmark; the judged metric is win rate, which lives in the leaderboard).
- **Newly stalled this week**: count of deals that crossed the 7-day no-activity threshold within this window, neutral delta. This is the movement headline that feeds the At-Risk Deals section.
### Section 5: Rep Leaderboard (Weekly)
- **One row per rep**, sorted by closed-won value this week descending: Deals Closed (count + value, WoW delta), Win Rate (judged color vs. that rep's trailing 8-week, WoW delta per the Global Delta Rule), Calls Logged (neutral delta), Emails Logged (neutral delta), Open Pipeline (value only, no delta; the per-rep WoW delta already renders in Section 3).
- The leaderboard is never averaged into a team row; comparing individuals is the point.
### Section 6: At-Risk Deals (Weekly)
- **Team-wide stalled count as the headline** (e.g. "12 deals stalled · $310K at risk" with WoW delta on the count), rendered as the card's internal KPI above the table.
- **Rep-level list**: days since last activity, owner, current stage, deal value, last logged interaction type; operational list, not a chart. 14-day stall flag is a fixed operational threshold, not a benchmark judgment.
### Section 7: Quota Pace by Rep (MTD + QTD)
- **One row per rep**: MTD attainment % with a mini progress bar, QTD attainment % with a mini progress bar, both in the judged pace color per the Judgment Rules. Sorted by QTD attainment ascending so the reps who need attention surface first. No WoW deltas anywhere in this section, per the Cadence Rule.
- **No team-blended attainment number**, ever. The section exists to show which reps are behind; a blended average hides that.
### Section 8: Coverage Ratio (QTD)
- **One aggregate KPI**: total open pipeline value ÷ remaining team quota for the quarter, rendered as "<X.X>x" in the judged color, with the benchmark stated inline in the card ("benchmark 3.0x") and a subline showing the math: "<open pipeline $> open pipeline · <remaining $> remaining team quota". Team-level only; never sliced per rep.
### Section 9: Sales Cycle & Time-in-Stage (QTD)
- **Average sales cycle length (closed deals, QTD)**: KPI, neutral, no delta.
- **Average time-in-stage**: one horizontal bar per open stage, bar length proportional to average days in that stage; stage name left; on the right, "<X> days avg". On the single worst stage bar (the longest), append the dragging rep inline in the same right-hand label (e.g. "31 days avg · slowest: Sarah Chen, 44 days"), attached directly to the bar it describes, never floated as separate text. This is the rep-level drill for a team-level headline.
---
## Visualization Guide
| Metric | Message type | Chart to use | Why this chart |
|--------|-------------|--------------|----------------|
| Team pipeline by stage **+** stage conversion rate | Comparison across a shared axis | **Composite pipeline visual**: horizontal stage funnel (bars sized by stage value) with the stage conversion column row directly below it, in the same card (exact spec below) | The bars show where the money sits; the conversion row below shows how deals flow between stages; each % sits inside the transition it describes, so it can't be misread as conversion from the start |
| Per-rep pipeline by stage | Comparison + composition | One horizontal stacked bar per rep, stage tints matching the team funnel, "<N> deals · <value>" + WoW delta right | Shows who owns the pipeline and its stage mix in one glance; mirrors the funnel's labeling so the two read as one system |
| Pipeline movement (new, won/lost, newly stalled) | Comparison | 3 KPI cards (won and lost share one card), neutral deltas per the Global Delta Rule | Fewer cards, faster scan; movement is a scan, not a study |
| Rep leaderboard | Ranked comparison | Table, one row per rep, sorted by closed-won value descending | Comparing named individuals across four metrics; a chart would force four separate charts |
| At-risk deals | Operational list | Headline count KPI inside the card + table sorted by days stalled descending | Actionable list; charts add no value here; the count alone is useless without the list attached |
| Per-rep quota pace (MTD + QTD) | Part-of-whole (progress), ranked | One row per rep with two mini progress bars (MTD, QTD) in the judged pace color, sorted by QTD ascending | Progress bars show gap-to-target at a glance; ascending sort puts the problem reps on top |
| Coverage ratio | Single number, judged | KPI card, "<X.X>x" in judged color, benchmark stated inline, math subline | A ratio against an explicit benchmark; color is earned because 3:1 is a real reference point |
| Time-in-stage | Comparison | One horizontal bar per stage, avg days, worst stage carries the dragging rep inline in its right label | Shows which stage drags the process; the inline rep name is the drill-down without a second chart |
| WoW delta | Modifier | Inline ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0 by default, Weekly-cadence elements only | The delta annotates the number it belongs to; never its own chart or section |
**Composite pipeline visual, exact layout (the two parts together count as one visual element):**
Part A, horizontal stage funnel:
- One horizontal bar per open stage, top to bottom in pipeline order, aggregated across the whole team.
- Bar length proportional to SUM(amount) in that stage; fill #3b82f6, with progressively lighter blue tints allowed down the funnel.
- Stage name at the left of each bar, 14px, #c4c8d0; at the right: deal count with the "deals" unit and value separated by a middle dot, followed by the WoW delta of that stage's total value per the Global Delta Rule (e.g. "23 deals · $412.6K ↑4.1%"), 14px, #c4c8d0. Never render the count without its unit.
- No conversion badges or percentages on these bars. Volume and its WoW delta only.
Part B, stage conversion columns, rendered directly below Part A in the same card:
- One equal-width column per stage transition, ordered left to right (e.g. Prospecting to Discovery, Discovery to Demo, Demo to Proposal, Proposal to Negotiation).
- Above each column: the transition label in the form "Discovery to Demo" (abbreviate stage names to keep labels short), 14px, #c4c8d0.
- Inside each column: the conversion % from the previous stage as the dominant number, bold, near-white, centered on the #3b82f6 fill.
- Below each column: destination stage deal count with the "deals" unit and value separated by a middle dot (e.g. "9 deals · $147.2K"), 14px, #c4c8d0.
- Between adjacent columns: a gray → arrow, vertically centered on the columns.
- Directly below the columns, one required caption: "Each % = conversion from the previous stage, not from the start of the funnel · trailing 90 days". This caption is the single allowed text row below a chart in the whole report.
- No ↑/↓ arrows anywhere on this visual: conversion is not a WoW delta.
**Chart rules:**
- **No duplicate bar groups for "this week vs. prior week."** Use one bar per category plus an inline delta per the Global Delta Rule.
- **Merge charts that share an axis** before adding a new one (see the composite pipeline visual above; its two stacked parts count as one visual element toward the cap).
- **Any derived or secondary number must be attached directly to the element it describes**: printed on or inside the column or bar, not floated as separate text nearby (this is why the dragging rep renders inside the worst time-in-stage bar's label). The stage conversion caption is the single exception to the no-text-row-below-a-chart rule.
- Every chart title states the finding, not the variable names.
- Color is either: (a) #c4c8d0 delta-only, or (b) green/amber/red per the Judgment Rules table. Never green/red on a metric without a listed benchmark (fixed categorical uses excepted: the stalled-deal flag and days-stalled cells).
- Labels ≤ 3 words where possible; stage transition labels may run longer, so abbreviate stage names instead of dropping the transition format.
- One dominant number per KPI card.
- If a metric VALUE cannot be computed, show "N/A" in the card (a Weekly delta slot with no prior data shows a plain hyphen "-" per the Global Delta Rule). Never leave a card blank or skip it.
- **Cap total distinct visual elements at 8 across the report.** The manager report ships exactly at the cap; there is no room to add a visual without merging or cutting one. The ledger: (1) composite team funnel, (2) per-rep stacked pipeline bars, (3) pipeline movement KPI row, (4) rep leaderboard table, (5) at-risk deals card (headline KPI + table), (6) quota pace by rep rows, (7) coverage ratio card, (8) time-in-stage bars.
---
## Data Source Mapping
| Metric | Source | Field / Object | Notes |
|--------|--------|----------------|-------|
| Per-rep quota target | HubSpot Goals | Goals API: each rep's quota for current month and quarter | Missing rep quota renders "N/A" in that rep's pace row; do not block |
| Team quota | Derived | SUM(per-rep quotas found) | Note the gap if any rep's quota is missing |
| Per-rep attainment (MTD/QTD) | HubSpot | SUM(deals.amount) WHERE stage = Closed Won AND close_date within period, GROUP BY owner | ÷ that rep's quota for the period |
| Expected pace benchmark | Derived | days elapsed ÷ days in period, per period (month, quarter) | Benchmark used to judge per-rep quota pace |
| Team pipeline by stage | HubSpot | COUNT and SUM(deals.amount) GROUP BY deal_stage, all team owners, current + prior 7-day snapshot | Open deals only; each bar's right label carries the WoW delta of that stage's value |
| Per-rep pipeline by stage | HubSpot | SUM(deals.amount) GROUP BY owner, deal_stage, current + prior 7-day snapshot | Feeds the stacked bars; per-rep WoW delta on total open value |
| Stage conversion rate | HubSpot | Deals advancing per stage ÷ deals entering per stage, 90-day window, team-wide | Different window; label it in the chart title and the required caption below the chart |
| New deals created this week | HubSpot | deals.create_date within current 7-day window, all team owners | |
| Closed won / lost this week | HubSpot | COUNT and SUM(deals.amount) WHERE stage IN (Closed Won, Closed Lost) AND close_date within current 7-day window | Combined into one KPI card |
| Newly stalled this week | HubSpot | deals whose last_activity_date crossed today - 7 within this window AND stage not closed | Feeds the movement KPI and the At-Risk headline |
| Deals stalled 7+ days | HubSpot | deals WHERE last_activity_date < today - 7 AND stage ≠ Closed Won AND stage ≠ Closed Lost, all team owners | The At-Risk table |
| Per-rep deals closed this week | HubSpot | COUNT and SUM(deals.amount), Closed Won, current 7-day window, GROUP BY owner | Leaderboard sort key |
| Per-rep win rate | HubSpot | COUNT(Closed Won) ÷ (COUNT(Closed Won) + COUNT(Closed Lost)), current 7-day window, GROUP BY owner | |
| Per-rep win rate, trailing 8-week | HubSpot | Same calculation, rolling 8-week window ending last week, GROUP BY owner | Benchmark for judging each rep's win rate |
| Per-rep calls / emails logged | HubSpot | Activity records, type = call / email, current 7-day window, GROUP BY owner | High-velocity/SMB motion signal |
| Coverage ratio | Derived | SUM(open pipeline amount, team) ÷ (team quota - team Closed Won QTD) | Judged vs. 3.0x |
| Avg sales cycle (closed, QTD) | HubSpot | AVG(close_date - create_date) for Closed Won + Closed Lost, QTD, team-wide | |
| Avg time-in-stage | HubSpot | AVG(days in stage) per stage from Stage History, trailing 90-day window, team-wide | Worst stage also computes MAX by owner for the inline dragging-rep label |
| Last activity type | HubSpot | Most recent activity record on each deal (type + date) | |
| WoW delta (Weekly metrics) | Derived | (current 7-day value - prior 7-day value) ÷ prior 7-day value × 100 | Rendered per the Global Delta Rule; Weekly cadence only |
---
## Report Structure
Render the header block first and the footer last (both specified in the Style Contract). Weekly-cadence sections render first (pipeline is the first thing a manager checks), Monthly/Quarterly sections after. Produce sections in this exact order:
1. **Summary: What's Going On** *(mandatory, always first, bulleted, not prose)*
   Two bullet groups:
   - **"Wins This Week"**: 2-4 bullets. Team achievements and positive movement vs. last week, with a number in each (e.g. "Team closed $86K across 6 deals (↑22.4%)"). This goes first; the report can't read as only a list of problems.
   - **"Focus For This Week"**: 2-4 bullets. Specific, actionable gaps naming reps and deals (e.g. "12 deals stalled, 7 on Sarah Chen's book; Meridian at 21 days is the largest at $48K"). Bullet counts don't need to be even; use however many genuinely apply.
   Each bullet ≤20 words and carries a number. No "significant," "up," or "trending" without a figure. Prior-week numbers may appear in Summary prose for context ("vs. 12 last week"), but the change itself always renders per the Global Delta Rule in parentheses, e.g. "16 deals advanced vs. 12 last week (↑33.3%)"; never a bare "up 33.3%" without the arrow. If there's genuinely no win to report, state a neutral fact instead of fabricating one (e.g., "Team pipeline held steady week over week").
2. **Team Pipeline by Stage** *(Weekly; the composite pipeline visual per the Visualization Guide: horizontal stage funnel with the stage conversion columns directly below it; this is the dominant focal point of the page)*
3. **Pipeline by Rep** *(Weekly; one stacked bar per rep, stage tints matching the funnel, "<N> deals · <value>" + WoW delta right, sorted by open value descending)*
4. **Pipeline Movement This Week** *(Weekly; 3 KPI cards in one row: New Deals Created; Closed Won / Closed Lost combined card with both counts and values; Newly Stalled. All neutral deltas per the Global Delta Rule.)*
5. **Rep Leaderboard** *(Weekly; table sorted by closed-won value descending: Rep, Deals Closed (count · value, WoW delta), Win Rate (judged color vs. own trailing 8-week, WoW delta), Calls Logged (neutral delta), Emails Logged (neutral delta), Open Pipeline (value only, no delta). Never add a team-average row.)*
6. **At-Risk Deals: Act Now** *(Weekly; headline KPI inside the card: "<N> deals stalled · <value> at risk" with WoW delta on the count; then the table sorted by days stalled descending: Deal Name, Rep, Stage, Value, Days Stalled, Last Touch)*
   Include every deal stalled 7+ days across the whole team. Last Touch shows the date only (e.g. "Jul 3, 2026"), no interaction-type suffix. Days Stalled renders in the bordered box per the Style Contract: amber for 7-13 days, red for over 14.
   Flag any deal where days stalled > 14 with a red left-border (fixed threshold, not a judged metric); rows under 14 days get no left border.
7. **Quota Pace by Rep** *(MTD + QTD; one row per rep, sorted by QTD attainment ascending: rep name, MTD mini progress bar + attainment %, QTD mini progress bar + attainment %, both in the judged pace color. No WoW deltas. No team-blended attainment number anywhere.)*
8. **Coverage Ratio** *(QTD; one KPI card: the ratio as "<X.X>x" in judged color with "benchmark 3.0x" inline and the math subline "<open pipeline> open pipeline · <remaining> remaining team quota". No WoW delta.)*
9. **Sales Cycle & Time-in-Stage** *(QTD/90-day; avg cycle length KPI, neutral, no delta; one bar per stage for avg days-in-stage; the worst stage bar carries the dragging rep inline in its right label, e.g. "31 days avg · slowest: Sarah Chen, 44 days")*
---
## Design Language
- **UI library:** Plain HTML + inline CSS (fully self-contained artifact, no external dependencies)
- **Visual style:** Bloomberg terminal × Metabase dark mode; data-dense, zero decoration
- **Punctuation, hard rule: no em dashes or en dashes anywhere in the rendered report.** Not in titles, subtitles, section headers, labels, captions, table cells, notes, bullets, or chips. Use colons, commas, periods, or the middle dot ( · ) separator instead. Plain hyphens are allowed; the ban is on em and en dashes. Date ranges use a plain hyphen (Jul 7-13). Missing VALUES render as "N/A"; a WoW delta slot with no prior data renders a plain hyphen "-" per the Global Delta Rule.
- **Number formatting:** dollar amounts ending in three zeros abbreviate with K ($280,000 renders as "$280K", $21,000 as "$21K"); non-round amounts render in full ($48,050, $33,635). The currency code appears exactly once, in the footer ("All figures in CAD"), never appended to individual numbers.
- **Background:** #0f1117
- **Surface / card background:** #1a1d27
- **Primary text:** #f0f0f0
- **Secondary / label text:** #c4c8d0; must meet **WCAG AA contrast (≥4.5:1)** against whatever it sits on. Verify against both page background and card background before finalizing.
- **Text rendered inside a colored bar/segment** (e.g., conversion columns, stacked bar segments) must contrast against *that segment's fill color*, not the page background; typically near-white on saturated fills. Check each fill color individually; don't reuse one text color across all of them by default.
- **Accent color:** #3b82f6 (blue): progress bars, active highlights, links, conversion column fills, funnel and stacked-bar stage fills (with lighter tints down the funnel)
- **Judged-good:** #22c55e (green), only on metrics listed in the Judgment Rules table
- **Judged-monitor:** #f59e0b (amber), only on metrics listed in the Judgment Rules table
- **Judged-bad:** #ef4444 (red), only on metrics listed in the Judgment Rules table (fixed categorical uses excepted: the stalled-deal flag and days-stalled cells)
- **Neutral delta (everything else):** ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0, never green/red
- **Stalled deal flag (14+ days):** #ef4444 left border on table row
- **Font:** one sans-serif stack for everything, including numbers: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif. Numbers are bold with font-variant-numeric: tabular-nums. No monospace, no webfonts, no external fonts.
- **Font size floor: 14px everywhere, no exceptions**, including chart annotations, badges, and conversion-rate text on columns. Nothing in the report may render smaller than 14px.
- **Number size:** 28px for North Star numbers; 18px for accessory metrics; 14px for labels and chart annotations
- **Layout:**
  - Full-width single column
  - KPI cards in a 3-up row max
  - Tables full-width with zebra striping (#1a1d27 / #1f2333)
  - Delta/judgment badge inline, right-aligned, on the metric itself
- **One dominant focal point:** the composite team pipeline visual (Section 2) is the largest element on the page.
- **Chart titles state the finding**, not the variable names.
- **8-visual-element cap** is a hard constraint, not a suggestion.
- **Before shipping, spot-check every text/background pairing used in the report against a contrast checker.** This is a required step, not implied by picking colors from this palette.
---
## Style Contract (reproduce this exact rendering every run)
The report must look identical from run to run, and identical in style to the rep-level weekly report. Do not restyle, reinterpret, or "improve" the visual design. Implement exactly:
- **Page:** background #0f1117, content max-width 1100px, 32px page padding, single column, 16px vertical gap between cards.
- **Section eyebrows:** every section opens with an eyebrow line sitting above its card, outside it: 14px, uppercase, letter-spacing 0.08em, #c4c8d0, middle-dot separators. Weekly sections carry the week (e.g. "PIPELINE OVERVIEW · WEEK OF JUL 7-13"); Monthly/Quarterly sections carry the period instead (e.g. "QUOTA PACE · MTD + QTD · Q3 2026", "COVERAGE · Q3 2026").
- **Header block:** eyebrow "WEEKLY TEAM REPORT · <quarter>"; manager name + team name 24px bold #f0f0f0; subline 14px #c4c8d0 "Week of <Jul 7-13, 2026> · <N> reps · Generated <date>". Top-right status chip: #1a1d27 pill, 1px border rgba(255,255,255,0.08), border-radius 999px, 8px colored status dot (green/amber/red per the coverage ratio judgment) + 14px text (e.g. "Coverage 3.4x, above benchmark").
- **Cards:** #1a1d27 background, 1px solid rgba(255,255,255,0.07) border, border-radius 10px, 20-24px padding. Card-internal section titles: 14px uppercase letterspaced #c4c8d0.
- **Summary card:** 3px #3b82f6 left border; two-column layout; group headers 14px uppercase: "✓ WINS THIS WEEK" in #22c55e, "⚑ FOCUS FOR THIS WEEK" in #f59e0b; bullets 14px #f0f0f0 with dim bullet glyphs.
- **KPI pattern:** label 14px #c4c8d0 above the number; number 28px (North Star) or 18px (accessory) bold #f0f0f0; WoW delta inline immediately after the number, 14px, per the Global Delta Rule, Weekly-cadence cards only.
- **Composite pipeline visual:** per the exact layout in the Visualization Guide. Funnel bars: 24-28px tall, rounded corners, fill #3b82f6 with lighter tints down the funnel, stage label left, "<N> deals · <value>" + WoW delta right, 14px #c4c8d0. Conversion columns below: fill #3b82f6, conversion % 18px bold near-white centered, transition labels above, "<N> deals · <value>" below, gray → arrows between, required caption underneath.
- **Per-rep stacked pipeline bars:** 24-28px tall, rounded corners, segments in the same stage tints as the funnel (a segment's tint = its stage's funnel tint, so the two visuals read as one system), rep name left 14px #c4c8d0, "<N> deals · <value>" + WoW delta right 14px #c4c8d0. A single stage legend row above the bars, 14px, pills per the pill spec, one per stage. No values printed inside segments narrower than their text.
- **Quota pace rows:** one row per rep: rep name left 14px #f0f0f0; two mini progress bars side by side (MTD, QTD), each 8px tall, border-radius 4px, track #262a36, fill in the judged pace color, the attainment % 14px bold immediately right of each bar in the same judged color, with a 14px #c4c8d0 "MTD" / "QTD" label above each bar column (labels rendered once as column headers, not per row).
- **Coverage Ratio card anatomy:** internal title "COVERAGE VS. REMAINING QUOTA"; dominant number "<X.X>x" 28px bold in the judged color; immediately right of it, 14px #c4c8d0 "benchmark 3.0x"; directly below, one 14px #c4c8d0 subline: "<open pipeline> open pipeline · <remaining> remaining team quota". No delta slot.
- **Progress bars (all):** 8px tall, border-radius 4px, track #262a36; fill #3b82f6 unless the bar renders a judged metric, in which case the judged color per the Judgment Rules.
- **Pills:** stage/type pills in tables and legends: #262a36 background, 4px radius, 14px text.
- **Days-stalled cells:** small bordered box; over 14 days: #ef4444 text and border on translucent red; 7-14 days: #f59e0b text and border on translucent amber.
- **Tables:** full-width; column headers 14px uppercase letterspaced #c4c8d0; zebra rows #1a1d27 / #1f2333; 12-14px cell padding; rows stalled over 14 days also get the 3px #ef4444 left border.
- **Footer:** hairline top border; left side "<Manager name> · <Team> · Week of <range> · <Quarter> · All figures in <currency>"; right side "Data: HubSpot · team-weekly-report v1.0"; both 14px #c4c8d0.
- If a styling question is not answered here, resolve it with the nearest token above. Never introduce a new color, font, size, or component style.
---
## Output Contract
- **File type:** Single self-contained HTML file (all CSS inline, no external scripts or fonts)
- **Delivery method:** Rendered as an artifact in the chat
- **Required sections checklist:**
  - [ ] Header block + status chip (per Style Contract, chip judged on coverage ratio)
  - [ ] Summary / What's Going On (mandatory, always first, bulleted: Wins This Week + Focus For This Week)
  - [ ] Team Pipeline by Stage (composite visual: funnel + conversion columns, arrows between columns, required caption)
  - [ ] Pipeline by Rep (stacked bars, stage tints matching the funnel)
  - [ ] Pipeline Movement This Week (3 cards, won/lost combined)
  - [ ] Rep Leaderboard (no team-average row)
  - [ ] At-Risk Deals: Act Now (headline count + full-team table with Rep column)
  - [ ] Quota Pace by Rep (MTD + QTD, no WoW deltas, no team blend)
  - [ ] Coverage Ratio (judged vs. 3.0x, benchmark stated inline)
  - [ ] Sales Cycle & Time-in-Stage (worst stage carries the dragging rep inline)
  - [ ] Footer (per Style Contract)
- **Tone:** Operational and exec-shareable; specific numbers, named reps, named deals, achievements included, no fluff. Written for the manager ("your team"), not first-person rep voice.
- **Punctuation:** zero em dashes and zero en dashes anywhere in the output, including titles, labels, and table cells. Missing values are "N/A"; a delta slot with no prior data to compare is a plain hyphen "-".
- **Reading level and length:** Summary is bulleted, not prose; 4-8 bullets total, ≤20 words each. Tables and cards are not subject to this limit, but no text anywhere renders below the 14px floor.
- **Total visual elements:** ≤8 across the whole report; this report ships exactly at 8 per the ledger in the Visualization Guide.
- **WoW display:** inline on every Weekly-cadence metric, formatted per the Global Delta Rule (arrow + one-decimal %), neutral #c4c8d0 by default. Monthly/Quarterly-cadence metrics (quota pace, coverage ratio, sales cycle / time-in-stage) never render a WoW delta or an arrow of any kind, per the Cadence Rule.
- **Style:** must match the Style Contract exactly, run to run.
---
## Fallback Rules
| Scenario | Claude should… |
|----------|----------------|
| No rep quotas found in HubSpot Goals at all | Stop. Ask the manager for the team quota and per-rep targets. The pace and coverage sections cannot generate without them. |
| One rep's quota missing | Render "N/A" in that rep's pace row with a note; compute team quota from the reps that have one and note the gap. Do not block. |
| No Closed Won deals this week | Show the Closed Won side of the movement card as $0 / 0 deals with a neutral delta. Do not skip the section. |
| A rep has 0 closed deals this week (won + lost) | Win Rate cell renders "N/A" (no decided deals to rate), not 0% and not 100%. |
| A rep's trailing 8-week win rate cannot be computed | Fall back to a neutral #c4c8d0 win rate cell for that rep. Do not guess a color. |
| Prior 7-day window returns no data | Show a plain hyphen "-" in every affected delta slot (it means "no previous data to compare") plus a one-line note: "No prior week data available." Missing VALUES in cells still render "N/A". |
| No genuine "win" to report this week | State a neutral factual bullet instead of fabricating a positive spin (e.g., "Team pipeline held flat week over week"). |
| No deals stalled 7+ days | Render the At-Risk headline as "0 deals stalled" and one line "No deals at risk this week." in place of the table. Do not skip the section. |
| A deal has no Last Activity Date | Treat as stalled from Create Date. Flag with note. |
| Stage conversion rate cannot be calculated | Show "N/A" in that column with note: "Fewer than 10 deals closed in the past 90 days." |
| Time-in-stage cannot be computed for a stage | Show that bar as "N/A". If the worst stage can't be determined, omit the dragging-rep label entirely; never guess. |
| A metric field is missing on a deal record | Show "N/A" in that cell. Never infer or fill in missing values. |
| HubSpot returns an error on any query | Note the error at the top of the affected section. Proceed with all other sections. |
---
## What This Skill Is NOT For
- Do not use em dashes or en dashes anywhere in the rendered report: no titles, labels, captions, cells, or notes.
- Do not express a WoW delta in any format other than the Global Delta Rule (arrow + one-decimal %). No "prior week was X", no bare percentages, no arrows without percentages.
- Do not render a WoW delta, or any ↑/↓ arrow, on a Monthly/Quarterly-cadence metric (quota pace, coverage ratio, sales cycle / time-in-stage). Cadence is a hard boundary.
- Do not put ↑/↓ arrows on percentages that are not WoW deltas (e.g., stage conversion rate, attainment %, the coverage ratio).
- Do not render a team-blended quota attainment number; attainment is rep-level only in this report.
- Do not render per-rep coverage ratios; coverage is team-level only (territory math, not an individual metric).
- Do not average the leaderboard into a team row; comparing named individuals is the point.
- Do not render the at-risk count without the rep-level deal list attached; the headline is useless without the list someone can act on.
- Do not show internal deal IDs anywhere in the report; always use deal names and rep names.
- Do not drop the horizontal stage funnel from Team Pipeline. The conversion columns render below it as a supplement, never as a replacement.
- Do not color-code direction as good/bad on any metric without a listed benchmark.
- Do not render two bar groups for "this week vs. prior week"; use the single-element-plus-inline-delta pattern.
- Do not float a derived rate or drill-down (like a conversion % or the dragging rep) as separate text near a chart; attach it to the column or bar it describes.
- Do not write the Summary as a paragraph; it must be two bulleted groups, wins first.
- Do not present the report as only a list of problems; it must show what's working before what's broken.
- Do not fabricate a "win" bullet if there isn't a real one; state a neutral fact instead.
- Do not render any text below the 14px floor, including chart annotations.
- Do not generate forecast models or predict close probability; use HubSpot's own probability field if needed.
- Do not include call transcripts, call quality coaching, or Fireflies content; that belongs to the rep-level report, not this one.
- Do not create new CRM records, update deal stages, or log activities.
- Do not filter to a single rep's deals; this report always covers every rep on the team. (A single-rep view is the rep-level weekly report's job.)
- Do not compare against periods other than those defined by the Cadence Rule unless explicitly asked.
- Do not fabricate missing fields; surface the gap instead.
- Do not exceed 8 total visual elements; if content doesn't fit, cut or merge, don't shrink the font.
- Do not deviate from the Style Contract: no new colors, fonts, sizes, or component styles.
---
## Example Trigger
> "Can you pull the team report? Pipeline review is at 10 and I want to know which reps and which deals to push on."
> "Run my team report."
> "How's the team pipeline looking this week?"
> "Which of my reps are behind on quota?"
