---
name: weekly-report
description: "Generates a weekly sales performance report (HTML artifact) covering quota attainment, pipeline health, activity volume, win rate, stalled deals, and WoW deltas from HubSpot, Fireflies, and Google Calendar. Shareable as-is with a manager."
version: 5.4
author: yuji-jeong
data_sources:
  - HubSpot
  - Fireflies
  - Google Calendar
trigger_phrases:
  - "run my weekly report"
  - "how's my pipeline looking"
  - "am I on track this quarter"
  - "pull my weekly report"
output_format: HTML artifact
---
# Weekly Sales Performance Report
## Goal
This report answers one question at the start of each week: am I moving in the right direction, and where should I spend the next hour? It is a **week-over-week directional tool**, not a statistically validated forecast. Small sample sizes are expected and fine. It is also written to be **shareable with a manager as-is**: it must show what's going well, not just what's broken. Target: a 60-second read that ends with the rep knowing what to do next.
---
## Prerequisites
- **HubSpot connector connected**: deal records, activity logs, pipeline data, and quota target all come from here.
- **Fireflies connector connected**: used to identify call type, link call context to deal movement, and surface call quality signal.
- **Google Calendar connector connected**: used for meetings booked (scheduled/created) this week, regardless of whether the contact showed up.
- **Quota target in HubSpot Goals**: the report cannot generate without this number. If it cannot be found, Claude must ask the rep to provide it before proceeding. Do not generate the report without it.
---
## North Star Metrics (one per question)
| # | Decision | North Star Metric |
|---|----------|-------------------|
| 1 | Am I on pace to hit quota? | QTD Quota Attainment % + $/week needed to hit target |
| 2 | Is my pipeline healthy enough? | Total open pipeline value + deals advanced vs. stalled this week |
| 3 | Am I doing enough activity? | Calls scheduled vs. connected + emails sent + meetings booked this week (all with WoW deltas) |
| 4 | Am I winning the deals I work? | Win rate this week, judged against my trailing average |
| 5 | Which deals need my attention right now? | List of stalled deals (no activity + no stage change in 7+ days) |
| 6 | Did I move up or down vs. last week? | WoW delta on every metric above (inline, on the metric itself, not a separate section) |
---
## Global Delta Rule (applies to every number in the report)
This is the single, universal format for week-over-week change. It applies to every metric, card, chart, and table cell. No exceptions.
- A WoW delta is always and only rendered as an arrow followed by the percentage to one decimal place: ↑16.7% or ↓4.2%.
- Never "prior week was X". Never a bare percentage without an arrow. Never an arrow without a percentage. Never "flat" or "=" in any delta slot; when a delta cannot be computed, the hyphen rule below applies.
- If the delta rounds to 0.0%, render ↑0.0% (zero or positive gets ↑, negative gets ↓).
- If a WoW delta cannot be computed because there is no prior data to compare (including the delta slot next to Projected Close at Pace), render a plain hyphen "-" in the delta slot. The hyphen means "no previous data to compare". Never "N/A" in a delta slot; "N/A" stays the indicator for missing VALUES in cells and cards.
- No suffix text on the delta itself (no "vs prior week", no "WoW"). Section eyebrows already carry the week context.
- ↑ and ↓ are reserved strictly for WoW deltas. A percentage that is not a WoW delta (stage conversion rate, attainment %, the win rate value itself, deal source split) never gets an arrow.
- Delta color: #c4c8d0 neutral by default. Green/amber/red only on the three judged metrics in the Judgment Rules table, and even then the format stays arrow + one-decimal percentage.
---
## Judgment Rules: When Color Means an Opinion, and When It's Just a Fact
**The default: direction is a fact, not an opinion.** Whether a number went up or down is objectively true. Whether "up" is *good* usually depends on context the report doesn't have. So by default, direction is neutral.
- **Neutral metrics (no benchmark exists):** delta rendered per the Global Delta Rule in #c4c8d0 next to the number. No green, no red. Applies to: total open pipeline value, new deals created, deals advanced, calls scheduled/connected, emails sent, meetings booked, average deal size, stalled deal count.
- **Judged metrics (a real benchmark exists):** the only places allowed to render an opinion, because there's a defensible number to compare against.
| Metric | Benchmark | Green (good) | Amber (monitor) | Red (bad) |
|---|---|---|---|---|
| Win rate | Rep's trailing 8-week win rate | At or above benchmark | Within 10 pts below | More than 10 pts below |
| Connect rate | Rep's trailing 8-week connect rate | At or above benchmark | Within 10 pts below | More than 10 pts below |
| Quota pace | Required $/week to hit target | Actual weekly pace ≥ required pace | 80-99% of required pace | Below 80% of required pace |
If a benchmark can't be computed (see Fallback Rules), fall back to a neutral #c4c8d0 delta. Never guess a color.
---
## Accessory Metrics
### Section 1: Quota Attainment
- **Closed-won deal count this week**: reveals whether attainment is volume- or size-driven.
- **Average deal size (won deals, QTD)**: context for pace, neutral delta only.
- **Weeks remaining in quarter**: used to compute required weekly pace (a judged metric).
### Section 2: Pipeline Health
- **New deals created this week**: neutral delta only.
- **Deals by stage + stage conversion rate**: one composite pipeline visual, two stacked parts in the same card. Part A: the horizontal stage funnel, one bar per open stage, bar length proportional to SUM(amount), stage name left; on the right, the deal count with its unit, the value, and that stage's WoW value delta (e.g. "8 deals · $98.7K ↑4.1%"). Never a bare count: "8" alone is ambiguous, it must read "8 deals". Part B: the stage conversion column row rendered directly below it (exact layout in the Visualization Guide). Conversion is COUNT(advanced) ÷ COUNT(entered) per stage over a trailing 90-day window, a different time window than the rest of the report; the conversion row must say so via a "(90-day)" tag in its title plus the required caption below it. Conversion percentages live only in Part B, never as badges on the funnel bars.
- **Deal source breakdown (won deals, QTD)**: compact pills inline in the KPI row, each pill "<X>% <Source> (<N> deals)". Include zero-share sources (e.g. "0.0% Referral (0 deals)"): a zero is a signal the rep should act on, not noise to hide. Neutral, no judgment.
### Section 3: Activity Volume
- **Calls scheduled / connected this week**: one combined KPI card. Neutral deltas on scheduled/connected.
- **Connect rate this week**: compact third line inside the Calls card, in its judged color per the Judgment Rules, with its WoW delta per the Global Delta Rule.
- **Emails sent this week**: neutral delta only.
- **Meetings booked this week**: neutral delta only.
- **Meetings confirmed for next week**: talking-point footnote at the bottom of the Emails & Meetings card, count plus attendee company/contact names in parentheses (e.g. "5 meetings next week already confirmed (Meridian, Riverstone, Foxglove, Thornfield, Kestrel)"). Gives the rep and manager concrete talking points.
### Section 4: Win Rate
- **Win rate this week**: judged against trailing 8-week average.
- **Average deal size, won vs. lost**: one bar per category with a tick marker for last week's value on the same bar. Neutral, no judgment.
- **Average sales cycle length**: neutral delta only.
### Section 5: Stalled Deals
- **Days since last activity, current stage, deal value, last logged interaction type**: operational list, not a chart. 14-day stall flag is a fixed operational threshold, not a benchmark judgment.
### Section 6: WoW Deltas (inline layer, not a standalone section)
- Applies the Global Delta Rule and the direction/judgment split to every metric, displayed inline on the metric itself.
---
## Fireflies Enrichment (supplementary, not blocking)
For each call that appears in Fireflies this week, surface:
- **Call type classification**: discovery, demo, follow-up, negotiation, or check-in, inferred from transcript content and meeting title.
- **Deal linkage**: match the call to an open HubSpot deal by company/contact name. Always compute this: it feeds the Advanced (48h)? determination and the stalled-deals table's Fireflies Call Type column. It does not get its own column in the calls table. Wherever a deal is referenced in report output, use the deal NAME, never an internal deal ID (reps don't know IDs).
- **Call-to-deal-movement correlation**: did the linked deal advance in the 48 hours following the call? If the rate is 0% or low across this week's calls, this must appear as a one-line callout in the Summary, not only in the table.
- **Call quality signal**: one short, specific line per call, always starting with the call duration (e.g. "38 min, ..."), then how the call went and what could improve next time, inferred from the transcript (e.g., talk-time ratio skewed toward the rep, no next step confirmed, prospect raised an unaddressed objection, pricing question deflected). This line also absorbs any 48h-window context that would otherwise clutter the Advanced (48h)? column (e.g. "deal closed won 4 days later, outside the 48h window"). Keep it concrete and coachable, not a vague sentiment score like "positive/negative." This is what makes the calls table useful for actually improving, not just tracking volume.
> If Fireflies is unavailable or returns no transcripts, skip this enrichment block entirely and note it at the top of the report. Do not block report generation.
---
## Visualization Guide
| Metric | Message type | Chart to use | Why this chart |
|--------|-------------|--------------|----------------|
| QTD Quota Attainment % | Part-of-whole (progress) | Horizontal progress bar (KPI card) | Single number with a target; progress bar shows gap at a glance |
| Projected close vs. quota, $/week needed | Relationship (pace vs. target) | KPI cards: projected close neutral with a plain hyphen "-" in its delta slot (no prior data to compare); required $/week in judged color per Judgment Rules | Color is earned only on required $/week (real benchmark); projected close is an extrapolation and stays neutral |
| Total open pipeline value | Single number | KPI card, neutral delta per Global Delta Rule | Scalar with no benchmark; delta only |
| Deals by stage **+** stage conversion rate | Comparison across a shared axis | **Composite pipeline visual**: horizontal stage funnel (bars sized by stage value) with the stage conversion column row directly below it, in the same card (exact spec below) | The bars show where the money sits; the conversion row below shows how deals flow between stages; each % sits inside the transition it describes, so it can't be misread as conversion from the start |
| Activity volume (calls, emails, meetings) | Comparison | 2 KPI cards (scheduled/connected share the Calls card with Connect Rate; emails + meetings share the second), per Report Structure item 4 | Fewer cards, faster scan |
| Win rate | Single number, judged | KPI card, judged color per Judgment Rules, WoW delta per Global Delta Rule | Rate alone loses context; color reflects vs.-benchmark, not raw direction |
| Won vs. lost average deal size | Comparison + WoW | One bar per category (won, lost), Won fill #3b82f6 and Lost fill #ef4444, a tick mark at last week's value with its label (e.g. "$21K prior") anchored directly at the tick, and an inline delta per the Global Delta Rule; no explainer caption under the bars | Replaces two-bar-groups-side-by-side; the label at the tick makes above-vs-below-prior readable at a glance |
| Stalled deals | Operational list | Table, sorted by days stalled descending | Actionable list; charts add no value here |
| Deal source breakdown | Part-of-whole | Compact pills inline in the KPI row, one per source, "<X>% <Source> (<N> deals)" | 3 slices only; compact enough to fold into the row above it |
| WoW delta | Modifier | Inline ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0 by default | The delta annotates the number it belongs to; never its own chart or section |
**Composite pipeline visual, exact layout (the two parts together count as one visual element):**
Part A, horizontal stage funnel (identical to the approved rendering):
- One horizontal bar per open stage, top to bottom in pipeline order.
- Bar length proportional to SUM(amount) in that stage; fill #3b82f6, with progressively lighter blue tints allowed down the funnel as in the approved rendering.
- Stage name at the left of each bar, 14px, #c4c8d0; at the right: deal count with the "deals" unit and value separated by a middle dot, followed by the WoW delta of that stage's total value per the Global Delta Rule (e.g. "8 deals · $98.7K ↑4.1%"), 14px, #c4c8d0. Never render the count without its unit.
- No conversion badges or percentages on these bars. Volume and its WoW delta only.
Part B, stage conversion columns, rendered directly below Part A in the same card:
- One equal-width column per stage transition, ordered left to right (e.g. Prospecting to Discovery, Discovery to Demo, Demo to Proposal, Proposal to Negotiation).
- Above each column: the transition label in the form "Discovery to Demo" (abbreviate stage names to keep labels short), 14px, #c4c8d0.
- Inside each column: the conversion % from the previous stage as the dominant number, bold, near-white, centered on the #3b82f6 fill.
- Below each column: destination stage deal count with the "deals" unit and value separated by a middle dot (e.g. "5 deals · $74.4K"), 14px, #c4c8d0.
- Between adjacent columns: a gray → arrow, vertically centered on the columns.
- Directly below the columns, one required caption: "Each % = conversion from the previous stage, not from the start of the funnel · trailing 90 days". This caption is the single allowed text row below a chart in the whole report.
- No ↑/↓ arrows anywhere on this visual: conversion is not a WoW delta.
**Chart rules:**
- **No duplicate bar groups for "this week vs. prior week."** Use one bar per category plus a marker tick for the prior value and an inline delta per the Global Delta Rule.
- **Merge charts that share an axis** before adding a new one (see the composite pipeline visual above; its two stacked parts count as one visual element toward the cap).
- **Any derived or secondary number (like a conversion rate) must be attached directly to the element it describes**: printed on or inside the column or bar, not floated as separate text nearby. The stage conversion caption is the single exception to the no-text-row-below-a-chart rule.
- Every chart title states the finding, not the variable names.
- Color is either: (a) #c4c8d0 delta-only, or (b) green/amber/red per the Judgment Rules table. Never green/red on a metric without a listed benchmark (fixed categorical uses excepted: the Lost bar fill, the stalled-deal flag, and days-stalled cells).
- Labels ≤ 3 words where possible; stage transition labels may run longer, so abbreviate stage names instead of dropping the transition format.
- One dominant number per KPI card.
- If a metric VALUE cannot be computed, show "N/A" in the card (a delta slot with no prior data shows a plain hyphen "-" per the Global Delta Rule). Never leave a card blank or skip it.
- **Cap total distinct visual elements at 8 across the report.** Merge or cut before rendering if a draft exceeds this.
---
## Data Source Mapping
| Metric | Source | Field / Object | Notes |
|--------|--------|----------------|-------|
| Quota target | HubSpot Goals | Goals API: rep quota for current quarter | If not found, stop and ask the rep before generating |
| Closed Won revenue (QTD) | HubSpot | deals.amount WHERE stage = Closed Won AND close_date within current quarter | |
| QTD deal count | HubSpot | COUNT(deals) WHERE stage = Closed Won AND close_date within current quarter | |
| Average deal size (QTD) | HubSpot | deals.amount, Closed Won, QTD | SUM ÷ COUNT |
| Projected close at pace | Derived | (Closed Won QTD ÷ weeks elapsed) × weeks remaining | Neutral display; this is an extrapolation, not a judgment |
| Required $/week to hit quota | Derived | (Quota - Closed Won QTD) ÷ weeks remaining | Benchmark used to judge quota pace |
| Total open pipeline value | HubSpot | SUM(deals.amount) WHERE stage ≠ Closed Won AND stage ≠ Closed Lost | |
| Deals advanced this week | HubSpot | deals where stage_change_date falls within current 7-day window | HubSpot Stage History property |
| Deals stalled 7+ days | HubSpot | deals WHERE last_activity_date < today - 7 AND stage ≠ Closed Won AND stage ≠ Closed Lost | |
| New deals created this week | HubSpot | deals.create_date within current 7-day window | |
| Deals by stage | HubSpot | COUNT and SUM(deals.amount) GROUP BY deal_stage, current + prior 7-day snapshot | Open deals only; each bar's right label carries the WoW delta of that stage's value |
| Stage conversion rate | HubSpot | Deals advancing per stage ÷ deals entering per stage, 90-day window | Different window; label it in the chart title and the required caption below the chart |
| Deal source breakdown | HubSpot | deals.deal_source (Inbound / Outbound / Referral), Closed Won, QTD | |
| Calls scheduled / connected this week | HubSpot | Activity records, type = call, current 7-day window | Combined into one KPI card |
| Emails sent this week | HubSpot | Activity records, type = email, within current 7-day window | |
| Meetings booked this week | Google Calendar | Events created within current 7-day window with ≥1 external attendee | Creation date, not occurrence date |
| Meetings confirmed next week | Google Calendar | Events occurring within the NEXT 7-day window with ≥1 external attendee; include attendee company/contact names | Occurrence date, not creation date; feeds the Emails & Meetings card footnote |
| Win rate | HubSpot | COUNT(Closed Won) ÷ (COUNT(Closed Won) + COUNT(Closed Lost)), current 7-day window | |
| Win rate, trailing 8-week average | HubSpot | Same calculation, rolling 8-week window ending last week | Benchmark for judging this week's win rate |
| Connect rate | HubSpot | Calls connected ÷ calls scheduled, current 7-day window | |
| Connect rate, trailing 8-week average | HubSpot | Same calculation, rolling 8-week window ending last week | Benchmark for judging this week's connect rate |
| Won vs. lost deal size | HubSpot | AVG(deals.amount) GROUP BY stage (Closed Won / Closed Lost), current + prior 7-day window | Prior week value used as chart tick marker, not a second bar group |
| Avg sales cycle (closed) | HubSpot | AVG(close_date - create_date) for Closed Won + Closed Lost, current 7-day window | |
| Last activity type | HubSpot | Most recent activity record on each deal (type + date) | |
| Call type classification | Fireflies | Transcript content + meeting title | Inferred: discovery / demo / follow-up / negotiation / check-in |
| Call-to-deal movement | HubSpot + Fireflies | Match Fireflies call to HubSpot deal; check stage_change_date within 48h post-call | Surface in Summary if 0% or low |
| Call quality signal | Fireflies | Transcript content + call duration | Starts with the duration; one concrete, coachable line per call, not a sentiment score |
| WoW delta (all metrics) | Derived | (current 7-day value - prior 7-day value) ÷ prior 7-day value × 100 | Rendered per the Global Delta Rule; color per Judgment Rules |
---
## Report Structure
Render the header block first and the footer last (both specified in the Style Contract). Produce sections in this exact order:
1. **Summary: What's Going On** *(mandatory, always first, bulleted, not prose)*
   Two bullet groups:
   - **"Wins This Week"**: 2-4 bullets. Achievements and positive movement vs. last week, with a number in each. This goes first because the report is shared with a manager; it can't read as only a list of problems.
   - **"Focus For Next Week"**: 2-4 bullets. Specific, actionable gaps (e.g., a named stalled deal, a low call-to-movement rate). Bullet counts don't need to be even; use however many genuinely apply.
   Each bullet ≤20 words and carries a number. No "significant," "up," or "trending" without a figure. Prior-week numbers may appear in Summary prose for context ("vs. 12 last week"), but the change itself always renders per the Global Delta Rule in parentheses, e.g. "16 deals advanced vs. 12 last week (↑33.3%)"; never a bare "up 33.3%" without the arrow. If there's genuinely no win to report, state a neutral fact instead of fabricating one (e.g., "Pipeline held steady week over week").
2. **Quota Attainment** *(dominant closed-won $ with WoW delta and the attainment % as the big blue companion number; progress bar: QTD % vs. 100%, right axis label K-abbreviated, e.g. "$280K"; projected close neutral with an "X% of quota" subline and a plain hyphen "-" in its delta slot, never "N/A"; required $/week in judged color, meaning both the dollar amount text itself and the green ✓ immediately to its right render in that judged color when on pace, e.g. "$19,825 ✓" with "$19,825" also in #22c55e, not left neutral/white; and directly below it a line in the same judged color, e.g. "Actual $33,635 · 70% above target" (or "below target"); closed-won deal count and avg size as the third sub-KPI; full anatomy in the Style Contract)*
3. **Pipeline Health** *(KPI row: open value, new deals, deals advanced, each with a neutral delta per the Global Delta Rule; one composite pipeline visual per the Visualization Guide: horizontal stage funnel with the stage conversion columns directly below it; deal source as a compact pill inline in the KPI row)*
4. **Activity This Week** *(2 KPI cards, keeping the minimal layout. Calls card: scheduled and connected numbers with neutral deltas per the Global Delta Rule, plus Connect Rate as a compact third line in its judged color with its WoW delta, never in the Win Rate section. Emails & Meetings card: emails sent and meetings booked with neutral deltas, plus the talking-point footnote at the bottom: "<N> meetings next week already confirmed (<attendee names>)", or "0 meetings next week confirmed so far" when none.)*
5. **Win Rate** *(KPI card, judged color, WoW delta per the Global Delta Rule, with a context line below the number: "2 won · 1 lost this week" plus "Trailing 8-week average: X%"; one bullet-bar chart for won vs. lost deal size with a prior-week tick marker whose label, e.g. "$21K prior", is anchored directly at the tick line (centered on it, just below it) so the reader instantly sees whether this week sits above or below the prior value, and an inline delta per the Global Delta Rule. Do not render a generic explainer caption like "Tick mark = prior week's average deal size"; the label at the tick makes it self-explanatory. A prior dollar VALUE label on a tick is allowed; a prior PERCENTAGE as text, like "prior week 50%", is not: WoW change only ever renders as arrow + one-decimal %.)*
6. **Stalled Deals: Act Now** *(table sorted by days stalled descending: Deal Name, Stage, Value, Days Stalled, Last Touch, Fireflies Call Type if available)*
   Include every deal stalled 7+ days. Last Touch shows the date only (e.g. "Jun 23, 2026"), no interaction-type suffix. Fireflies Call Type shows the type only (e.g. "Negotiation"), "N/A" when no call. Days Stalled renders in the bordered box per the Style Contract: amber for 7-13 days, red for over 14.
   Flag any deal where days stalled > 14 with a red left-border (fixed threshold, not a judged metric); rows under 14 days get no left border.
7. **Calls This Week (Fireflies)** *(supplementary table with exactly four columns: Company, Call Type, Advanced (48h)?, Call Quality Signal. No Call-name column and no Deal Linked column: both are redundant. Advanced (48h)? cells contain only "Yes", "No", or "N/A" with nothing appended after them; any context, like a deal closing 4 days post-call outside the 48h window or the deal being already closed, belongs in the Call Quality Signal text instead.)*
   Only render if Fireflies data is available; omit silently and note it in the Summary otherwise.
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
- **Text rendered inside a colored bar/segment** (e.g., conversion columns, stat fills) must contrast against *that segment's fill color*, not the page background; typically near-white on saturated fills. Check each fill color individually; don't reuse one text color across all of them by default.
- **Accent color:** #3b82f6 (blue): progress bars, active highlights, links, conversion column fills
- **Judged-good:** #22c55e (green), only on metrics listed in the Judgment Rules table
- **Judged-monitor:** #f59e0b (amber), only on metrics listed in the Judgment Rules table
- **Judged-bad:** #ef4444 (red), only on metrics listed in the Judgment Rules table (fixed categorical uses excepted: the Lost bar fill, the stalled-deal flag, and days-stalled cells)
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
- **One dominant focal point:** QTD Attainment progress bar is the largest element on the page.
- **Chart titles state the finding**, not the variable names.
- **8-visual-element cap** is a hard constraint, not a suggestion.
- **Before shipping, spot-check every text/background pairing used in the report against a contrast checker.** This is a required step, not implied by picking colors from this palette.
---
## Style Contract (reproduce this exact rendering every run)
The report must look identical from run to run. Do not restyle, reinterpret, or "improve" the visual design. Implement exactly:
- **Page:** background #0f1117, content max-width 1100px, 32px page padding, single column, 16px vertical gap between cards.
- **Section eyebrows:** every section opens with an eyebrow line sitting above its card, outside it: 14px, uppercase, letter-spacing 0.08em, #c4c8d0, middle-dot separators (e.g. "QUOTA ATTAINMENT · Q3 2026", "PIPELINE OVERVIEW · WEEK OF JUL 7-13").
- **Header block:** eyebrow "WEEKLY SALES REPORT · <quarter>"; rep name 24px bold #f0f0f0; subline 14px #c4c8d0 "Week of <Jul 7-13, 2026> · Generated <date>". Top-right status chip: #1a1d27 pill, 1px border rgba(255,255,255,0.08), border-radius 999px, 8px colored status dot (green/amber/red per the quota pace judgment) + 14px text (e.g. "Quota pacing above target").
- **Cards:** #1a1d27 background, 1px solid rgba(255,255,255,0.07) border, border-radius 10px, 20-24px padding. Card-internal section titles: 14px uppercase letterspaced #c4c8d0.
- **Summary card:** 3px #3b82f6 left border; two-column layout; group headers 14px uppercase: "✓ WINS THIS WEEK" in #22c55e, "⚑ FOCUS FOR NEXT WEEK" in #f59e0b; bullets 14px #f0f0f0 with dim bullet glyphs.
- **KPI pattern:** label 14px #c4c8d0 above the number; number 28px (North Star) or 18px (accessory) bold #f0f0f0; WoW delta inline immediately after the number, 14px, per the Global Delta Rule.
- **Activity cards anatomy:** two cards, minimal layout. Calls card: label "Calls Scheduled / Connected"; the scheduled number with a small "scheduled" label and its delta, the connected number with a small "connected" label and its delta, stacked; below them, Connect Rate as a compact third line, 14px, the rate in its judged color with its WoW delta. Emails & Meetings card: "Emails Sent" and "Meetings Booked" numbers with their deltas; at the bottom of the card, the talking-point footnote, 14px #c4c8d0: "<N> meetings next week already confirmed (<attendee names>)", or "0 meetings next week confirmed so far" when none. The footnote lives inside the KPI card; it is not a below-chart caption, so the stage-conversion caption remains the single allowed text row below a chart.
- **Progress bar:** 8px tall, border-radius 4px, fill #3b82f6, track #262a36; axis line below: left "0", center "Day X of Y", right the target amount K-abbreviated per the number formatting rule (e.g. "$280K", never "$280,000 CAD"), all 14px #c4c8d0.
- **Quota Attainment card anatomy:** internal title "QTD PROGRESS: $<target> TARGET" with the target K-abbreviated per the number formatting rule (e.g. "QTD PROGRESS: $280K TARGET"; no currency code, that lives in the footer). Dominant number: QTD closed-won $ (28px, "closed won" label, WoW delta inline); attainment % 28px #3b82f6 right-aligned on the same row. Progress bar and axis below. Then a 3-up sub-KPI row: (1) Projected Close at Pace with subline "<X>% of quota · neutral estimate"; its delta slot renders a plain hyphen "-" (no previous data to compare), never "N/A" (it is an extrapolation, not a tracked weekly metric); (2) Required $/Week to Hit Quota in the judged color, meaning the dollar amount text itself is rendered in that judged color (not left neutral/white), with the green ✓ immediately to the RIGHT of the number when judged green (e.g. "$19,825 ✓" with "$19,825" also in #22c55e), and DIRECTLY BELOW it a line in the same judged color: "Actual $33,635 · 70% above target" (or "below target"); (3) Closed Won Deals / Avg Size (QTD).
- **Won/Lost deal size bars:** Won fill #3b82f6, Lost fill #ef4444. These are fixed categorical fills from the approved rendering, not judgment colors; the Won bar is never green. White tick mark at the prior week value; its 14px #c4c8d0 label stating the prior dollar amount (e.g. "$21K prior") is horizontally centered on the tick line and placed directly below it, so it reads as attached to that exact position (position is the pointer: no triangle or arrow glyphs toward the tick). Current value and WoW delta right-aligned at the end of the row. No "Tick mark = ..." explainer caption under the bars; the stage-conversion caption in Pipeline Health remains the single allowed text row below a chart.
- **Composite pipeline visual:** per the exact layout in the Visualization Guide. Funnel bars: 24-28px tall, rounded corners, fill #3b82f6 with lighter tints down the funnel, stage label left, "<N> deals · <value>" + WoW delta right, 14px #c4c8d0. Conversion columns below: fill #3b82f6, conversion % 18px bold near-white centered, transition labels above, "<N> deals · <value>" below, gray → arrows between, required caption underneath.
- **Pills:** stage/type pills in tables: #262a36 background, 4px radius, 14px text.
- **Days-stalled cells:** small bordered box; over 14 days: #ef4444 text and border on translucent red; 7-14 days: #f59e0b text and border on translucent amber.
- **Tables:** full-width; column headers 14px uppercase letterspaced #c4c8d0; zebra rows #1a1d27 / #1f2333; 12-14px cell padding; rows stalled over 14 days also get the 3px #ef4444 left border.
- **Footer:** hairline top border; left side "Rep name · Week of <range> · <Quarter> · All figures in <currency>"; right side "Data: HubSpot · Google Calendar · Fireflies · weekly-report v5.4"; both 14px #c4c8d0.
- If a styling question is not answered here, resolve it with the nearest token above. Never introduce a new color, font, size, or component style.
---
## Output Contract
- **File type:** Single self-contained HTML file (all CSS inline, no external scripts or fonts)
- **Delivery method:** Rendered as an artifact in the chat
- **Required sections checklist:**
  - [ ] Header block + status chip (per Style Contract)
  - [ ] Summary / What's Going On (mandatory, always first, bulleted: Wins This Week + Focus For Next Week)
  - [ ] Quota Attainment (blocks on missing quota target)
  - [ ] Pipeline Health (horizontal stage funnel + stage conversion columns directly below it, arrows between columns, required caption)
  - [ ] Activity This Week (2 cards, not 4)
  - [ ] Win Rate (bullet-bar chart, not paired bar groups; WoW shown only per the Global Delta Rule, never as "prior week X%")
  - [ ] Stalled Deals: Act Now
  - [ ] Calls This Week (Fireflies), exactly four columns (Company, Call Type, Advanced (48h)?, Call Quality Signal), quality signal starts with the call duration; conditional on availability
  - [ ] Footer (per Style Contract)
- **Tone:** Operational and manager-shareable; specific numbers and actions, achievements included, no fluff. First-person where natural.
- **Punctuation:** zero em dashes and zero en dashes anywhere in the output, including titles, labels, and table cells. Missing values are "N/A"; a delta slot with no prior data to compare is a plain hyphen "-".
- **Reading level and length:** Summary is bulleted, not prose; 4-8 bullets total, ≤20 words each. Tables and cards are not subject to this limit, but no text anywhere renders below the 14px floor.
- **Total visual elements:** ≤8 across the whole report.
- **WoW display:** inline on every metric, formatted per the Global Delta Rule (arrow + one-decimal %). Neutral #c4c8d0 by default; colored judgment only for the three benchmarked metrics. Any delta slot with no prior data to compare renders a plain hyphen "-" (never "N/A"); Projected Close at Pace always renders "-" in its delta slot because it is an extrapolation with no prior-week comparison.
- **Style:** must match the Style Contract exactly, run to run.
---
## Fallback Rules
| Scenario | Claude should… |
|----------|----------------|
| Quota target not found in HubSpot Goals | Stop. Ask the rep for their quota. Do not generate any part of the report until provided. |
| No Closed Won deals this week | Show KPI card as $0 / 0 deals with a neutral delta. Do not skip the section. |
| No Closed Lost deals this week | Show win rate as "100% (0 losses this week)" with a clarifying note. |
| Prior 7-day window returns no data | Show a plain hyphen "-" in every affected delta slot (it means "no previous data to compare") plus a one-line note: "No prior week data available." Missing VALUES in cells still render "N/A". |
| Trailing 8-week average cannot be computed | Fall back to a neutral #c4c8d0 delta for that metric. Do not guess a color. |
| No genuine "win" to report this week | State a neutral factual bullet instead of fabricating a positive spin (e.g., "Pipeline held flat week over week"). |
| Fireflies unavailable or returns no transcripts | Note at top of Summary. Skip Section 7 entirely. Do not block report generation. |
| Google Calendar unavailable | Show "Meetings booked" as "N/A" with note; omit the next-week confirmed meetings footnote entirely. Proceed with remaining sections. |
| A deal has no Last Activity Date | Treat as stalled from Create Date. Flag with note. |
| Stage conversion rate cannot be calculated | Show "N/A" in that column with note: "Fewer than 10 deals closed in the past 90 days." |
| A metric field is missing on a deal record | Show "N/A" in that cell. Never infer or fill in missing values. |
| HubSpot returns an error on any query | Note the error at the top of the affected section. Proceed with all other sections. |
---
## What This Skill Is NOT For
- Do not use em dashes or en dashes anywhere in the rendered report: no titles, labels, captions, cells, or notes.
- Do not express a WoW delta in any format other than the Global Delta Rule (arrow + one-decimal %). No "prior week was X", no bare percentages, no arrows without percentages.
- Do not put ↑/↓ arrows on percentages that are not WoW deltas (e.g., stage conversion rate).
- Do not append anything after "Yes", "No", or "N/A" in the Advanced (48h)? column; that context belongs in the Call Quality Signal.
- Do not show internal deal IDs anywhere in the report; always use deal names.
- Do not drop the horizontal stage funnel from Pipeline Health. The conversion columns render below it as a supplement, never as a replacement.
- Do not color-code direction as good/bad on any metric without a listed benchmark.
- Do not render two bar groups for "this week vs. prior week"; use the single-bar-plus-marker pattern.
- Do not float a derived rate (like a conversion %) as separate text near a chart; attach it to the column or bar it describes.
- Do not write the Summary as a paragraph; it must be two bulleted groups, wins first.
- Do not present the report as only a list of problems; it's shared with a manager and must show what's working.
- Do not fabricate a "win" bullet if there isn't a real one; state a neutral fact instead.
- Do not render any text below the 14px floor, including chart annotations.
- Do not generate forecast models or predict close probability; use HubSpot's own probability field if needed.
- Do not summarize email threads or call recordings beyond call type and the call quality signal specified above.
- Do not create new CRM records, update deal stages, or log activities.
- Do not compare against periods other than the current 7-day window and the prior 7-day window unless explicitly asked.
- Do not fabricate missing fields; surface the gap instead.
- Do not include deals from other reps' pipelines; filter to the logged-in HubSpot user's owned deals only.
- Do not exceed 8 total visual elements; if content doesn't fit, cut or merge, don't shrink the font.
- Do not deviate from the Style Contract: no new colors, fonts, sizes, or component styles.
---
## Example Trigger
> "Hey, can you pull my weekly report? I have my 1:1 in an hour and want to know if I'm on pace and which deals I should flag."
> "Run my weekly report."
> "How's my pipeline looking this week?"
