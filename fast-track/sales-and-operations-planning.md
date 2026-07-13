---
name: sales-ops-planning-review
description: "Generates a Sales & Operations Planning (S&OP) review (HTML artifact) for an exec at a physical goods company covering net sales vs. target, demand forecast vs. supply position, sell-through and inventory turns, backorders and stockouts, days on hand, gross margin and AOV trend, plus a stockout-risk and supplier-delay flag. Shareable as-is with the leadership team."
version: 1.0
author: yuji-jeong
data_sources:
  - "ERP: SAP S/4HANA or NetSuite (orders, returns, net sales, targets, margin)"
  - "Inventory/ops tool: Cin7, Fishbowl, or Blue Yonder (stock levels, sell-through, days on hand)"
  - "Planning tool: Kinaxis RapidResponse or SAP IBP (demand forecast, supply position)"
  - "BI layer: Power BI or Tableau (optional exception layer for flags; only if connected)"
trigger_phrases:
  - "run my S&OP review"
  - "how's inventory looking this week"
  - "can supply cover the forecast"
  - "pull the sales and ops report"
  - "are we at risk of stockouts"
output_format: HTML artifact
---
# Sales & Operations Planning Review: Physical Goods
## Goal
This report answers two questions at the start of each cycle: are we hitting the sales number, and can supply actually meet the demand we're forecasting. It is a **directional planning tool**, not a statistically validated forecast; weekly figures on slower-moving SKUs will be noisy and that's fine. It is written to be **shareable with the leadership team as-is**: it must show what's going well, not just what's at risk. It surfaces exceptions, not a SKU-level inventory dump. Target: a 60-second read that ends with the exec knowing whether sales and supply are on plan and which one or two items need a decision now.
---
## Prerequisites
- **ERP connector connected (SAP S/4HANA or NetSuite)**: the default source for net sales, units, returns, new orders, backorders, margin, and AOV. Whichever ERP is connected is the source of truth; the tools named per metric below are typical defaults, not hard requirements.
- **Inventory/ops tool connected (Cin7, Fishbowl, or Blue Yonder)**: stock levels, sell-through, inventory turns, and days on hand. If none is connected but the ERP carries inventory data, read from the ERP and note the substitution in the affected section.
- **Planning tool connected (Kinaxis RapidResponse or SAP IBP)**: demand forecast and projected supply position. If no planning tool is connected, the Forecast vs. Supply section renders N/A with a note; it is never fabricated from sales history.
- **Monthly net sales target set in the ERP (company-wide, post-returns)**: the report cannot generate without this number. If it cannot be found, Claude must ask the exec for it before proceeding. Do not generate the report without it.
- **Days-on-hand target band (min-max days, e.g. 30-45)**: read from the planning tool or ERP if set. If it cannot be found, ask the exec once; if not provided, render days on hand neutral with a note rather than blocking.
- **Currency**: read from the currency field on ERP order records. If records show a single consistent currency, use it. If mixed or undeterminable, ask the exec which currency to report in before generating. Do not guess.
---
## Cadence Rule (which window each metric lives in)
Every metric in this report belongs to exactly one primary cadence. The cadence controls the time window, the delta treatment, and the section eyebrow.
- **Weekly metrics** (current 7-day window, WoW delta per the Global Delta Rule): backorders and stockouts vs. new orders, sell-through rate on fast-moving SKUs, days on hand (operational tracking), net sales this week (tracking line), and the flags.
- **Monthly metrics** (calendar month window, MoM delta per the Global Delta Rule where a prior month exists): net sales MTD vs. monthly target (the primary reported figure), demand forecast vs. supply position (the core S&OP cycle), gross margin %, AOV, inventory turns portfolio rollup, and the monthly average days on hand (the reported exec figure).
- **Dual-tracked metrics** (net sales, days on hand): the monthly figure is the headline; the weekly figure appears as one clearly labeled subline inside the same card with its own WoW delta. Never blend the two windows into a single number or a single delta slot.
- **Weekly exception check on forecast vs. supply**: the Forecast vs. Supply card is monthly, but if the current week's data shows supply coverage dropping below 100% of forecasted demand mid-cycle, render one amber callout line above the card content noting the gap. This is the only weekly element allowed inside that monthly card.
- Never mix cadences inside one visual element beyond the labeled-subline and exception-callout patterns above. A weekly element carries WoW deltas; a monthly element carries MoM deltas; neither borrows the other's window.
---
## North Star Metrics (one per decision)
| # | Decision | North Star Metric | Cadence |
|---|----------|-------------------|---------|
| 1 | Are we hitting the sales number? | Net sales (units × price, post-returns) MTD vs. monthly target | Monthly (primary) + Weekly tracking line |
| 2 | Can supply meet forecasted demand? | Supply coverage: projected supply ÷ forecasted demand for the current cycle | Monthly + weekly exception check |
| 3 | Are we losing sales to supply gaps right now? | Backorder $ exposure and stockout SKU count vs. new orders | Weekly |
| 4 | Is inventory converting to sales at a healthy pace? | Sell-through rate on fast movers + inventory turns portfolio rollup | Weekly + Monthly rollup |
| 5 | Are we overstocked or understocked? | Inventory days on hand vs. target band | Weekly ops + Monthly exec figure |
| 6 | Is profitability holding? | Gross margin % and AOV trend | Monthly |
| 7 | What needs a decision now? | Largest stockout risk ($ sales impact) + largest fulfillment/supplier delay | Weekly (point-in-time) |
| — | Did we move up or down vs. the prior period? | PoP delta on every metric above, per its own cadence per the Cadence Rule (inline on the metric itself, not a separate section) | — |
---
## Net, Not Gross: Returns Are Always Baked In
Net sales in this report is always **units × price minus returns and refund credits** for the window. Returned physical inventory affects both revenue and stock position, so gross bookings alone misstate the business. Net sales and gross sales are never blended or silently substituted for each other. If returns data cannot be read from the ERP for the window, render gross sales instead and state that plainly as a one-line note under the Net Sales card ("Returns data unavailable: figures shown are gross sales."), never silently.
---
## Global Delta Rule (applies to every PoP number in the report)
This is the single, universal format for period-over-period change: WoW on Weekly-cadence metrics, MoM on Monthly-cadence metrics. It applies to every metric, card, chart, and table cell it touches. No exceptions.
- A PoP delta is always and only rendered as an arrow followed by the percentage to one decimal place: ↑16.7% or ↓4.2%. The window it spans (WoW or MoM) is whatever that metric's cadence resolves to per the Cadence Rule; the rendered format is identical either way.
- Never "prior week was X" or "last month was X". Never a bare percentage without an arrow. Never an arrow without a percentage. Never "flat" or "=" in any delta slot; when a delta cannot be computed, the hyphen rule below applies.
- If the delta rounds to 0.0%, render ↑0.0% (zero or positive gets ↑, negative gets ↓).
- If a PoP delta cannot be computed because there is no prior data to compare, render a plain hyphen "-" in the delta slot. The hyphen means "no previous data to compare". Never "N/A" in a delta slot; "N/A" stays the indicator for missing VALUES in cells and cards.
- No suffix text on the delta itself (no "vs prior week", no "WoW"/"MoM"). Section eyebrows already carry the period context.
- ↑ and ↓ are reserved strictly for PoP deltas. A percentage that is not a PoP delta (attainment %, supply coverage %, sell-through rate, gross margin %, returns rate) never gets an arrow.
- Delta color: #c4c8d0 neutral by default. Green/amber/red only on the metrics listed in the Judgment Rules table, and even then the format stays arrow + one-decimal percentage.
---
## Judgment Rules: When Color Means an Opinion, and When It's Just a Fact
**The default: direction is a fact, not an opinion.** Whether a number went up or down is objectively true. Whether "up" is *good* usually depends on context the report doesn't have. So by default, direction is neutral.
- **Neutral metrics (no benchmark exists):** delta rendered per the Global Delta Rule in #c4c8d0 next to the number. No green, no red. Applies to: net sales $ values and units, returns rate, new orders $, backorder $ exposure, stockout SKU count, sell-through rate, inventory turns, gross margin %, AOV, and both flags.
- **Judged metrics (a real benchmark exists):** the only places allowed to render an opinion, because there's a defensible number to compare against.

| Metric | Benchmark | Green (good) | Amber (monitor) | Red (bad) |
|---|---|---|---|---|
| Net sales MTD pacing | Required $/day to hit monthly target, given days elapsed | Actual daily pace at or above required pace | 80-99% of required pace | Below 80% of required pace |
| Supply coverage % | 100% of forecasted demand for the current cycle | At or above 100% | 90-99.9% | Below 90% |
| Days on hand | Target band (min-max days) from Prerequisites | Within the band | Outside the band by up to 20% of the band width, either direction | Outside the band by more than 20%, either direction |

Days on hand is judged **in both directions**: too high ties up cash in stock, too low risks stockouts. Both extremes earn amber/red; there is no "good" direction, only the band. Gross margin % and AOV are intentionally **not** judged: they are lagging, slower-moving trend metrics with no hard external threshold in this report, and coloring them would invite reacting to promotion noise. If a benchmark can't be computed (see Fallback Rules), fall back to a neutral #c4c8d0 rendering. Never guess a color.
---
## Accessory Metrics
### North Star 1: Net Sales vs. Target
- **Net sales this week**: the weekly tracking line, neutral WoW delta, labeled subline inside the monthly card.
- **Units sold MTD**: neutral, supports the units × price story.
- **Returns $ and returns rate MTD** (returns $ ÷ gross sales): neutral; the rate never gets an arrow.
- **Required $/day to hit monthly target**: (monthly target - net sales MTD) ÷ days remaining in month. Benchmark used to judge MTD pacing.
- **Projected month-end at pace**: (net sales MTD ÷ days elapsed) × days in month. Neutral extrapolation, not a judgment; its delta slot always renders a plain hyphen "-".
### North Star 2: Forecast vs. Supply
- **Forecasted demand (current cycle)**: consensus demand plan from the planning tool, in $ or units per what the tool reports. Neutral.
- **Projected supply position**: on-hand inventory plus confirmed inbound receipts landing within the cycle. Neutral.
- **Supply coverage %** (projected supply ÷ forecasted demand): the judged headline of this card.
- **Weekly exception callout**: rendered only when the current week's check shows coverage below 100%; one amber line, factual, not a second judged number.
### North Star 3: Backorders and Stockouts
- **Backorder $ exposure**: value of open order lines unfulfilled past promise date. Neutral WoW delta, the headline of this card.
- **Stockout SKU count**: SKUs with zero available stock and open demand. Neutral WoW delta.
- **New orders $ this week**: neutral WoW delta; the comparison anchor showing whether supply gaps are growing faster than demand.
### North Star 4: Sell-Through and Turns
- **Sell-through rate, fast movers** (units sold ÷ units available at start of window, top SKUs by trailing-month revenue): weekly, neutral; the rate never gets an arrow, its WoW delta does.
- **Inventory turns, full portfolio** (annualized COGS ÷ average inventory value): monthly rollup, plain labeled line inside the card with its MoM delta.
### North Star 5: Days on Hand
- **Current days on hand** (on-hand units ÷ average daily units sold, trailing 4 weeks): weekly operational figure, judged against the target band.
- **Monthly average days on hand**: the reported exec figure, labeled subline with its MoM delta.
### North Star 6: Margin and AOV
- **Gross margin %** ((net sales - COGS) ÷ net sales, calendar month): neutral MoM delta on the value; the % itself never gets an arrow confusion, only the MoM delta does.
- **Average order value** (net sales ÷ order count, calendar month): neutral MoM delta.
### North Star 7: Flags
- **Largest stockout risk**: among SKUs where projected supply is below forecasted demand, flag by **$ sales impact** (forecasted units at risk × price), not by SKU count or strategic importance. One flag only, the single largest. Include product name, $ sales impact, and the specific signal (e.g. "projected supply covers 12 of 30 forecast days").
- **Largest fulfillment/supplier delay**: the single late purchase order or inbound shipment with the largest $ value. Include supplier or PO reference, $ value, and days late. One flag only.
---
## Visualization Guide
| Metric | Message type | Chart to use | Why this chart |
|--------|-------------|--------------|----------------|
| Net sales MTD vs. target + pacing | Part-of-whole (progress) | One composite card: horizontal progress bar (MTD attainment % vs. 100%), net sales MTD as the dominant number with MoM delta, this-week tracking subline with WoW delta, returns note, pacing sub-KPI row below (required $/day in judged color, projected month-end neutral) | Single dominant card matches the report's biggest decision; sublines keep weekly tracking and returns legible without extra charts |
| Supply coverage % | Single number, judged | KPI card, "<XXX>%" in judged color, benchmark stated inline, math subline showing projected supply and forecasted demand, amber weekly-exception callout only when triggered | A ratio against an explicit 100% benchmark; color is earned |
| Backorders/stockouts vs. new orders | Comparison + WoW | One bar per category (New Orders fill #3b82f6, Backorders fill #ef4444), backorder $ exposure headline with WoW delta above the bars, stockout SKU count as a labeled line below | Two categories only; bar-per-category plus a headline number reads faster than a grouped chart |
| Sell-through + turns | Comparison | KPI pair in one card: fast-mover sell-through rate with its WoW delta, portfolio turns rollup line below with its MoM delta, each labeled with its window | Two scalars telling one conversion story; a chart adds no value |
| Days on hand vs. band | Single number, judged | KPI card: current DOH 28px in judged color with the band stated inline ("target 30-45 days"), monthly average subline with MoM delta | A number against a band; the band text is the context, not a gauge graphic |
| Margin % + AOV | Comparison | One 2-up KPI row, each with a neutral MoM delta and "calendar month" window note | Two lagging scalars on the same window; a KPI row is faster to scan than trend charts |
| Flags: stockout risk + supplier delay | Operational | Small card, two labeled lines, no chart | Two named exceptions with dollar amounts; a chart adds no value |
| PoP delta | Modifier | Inline ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0 by default | The delta annotates the number it belongs to; never its own chart or section |
**Chart rules:**
- **No duplicate bar groups for "this period vs. prior period."** Use one bar per category plus an inline delta per the Global Delta Rule.
- **Merge charts that share an axis** before adding a new one; do not split sell-through and turns into separate cards, and do not split pacing out of the Net Sales card.
- **Any derived or secondary number must be attached directly to the element it describes**: printed on or inside the bar or line it belongs to, never floated as separate text nearby.
- Every chart title states the finding, not the variable names.
- Color is either: (a) #c4c8d0 delta-only, or (b) green/amber/red per the Judgment Rules table. Never green/red on a metric without a listed benchmark (the Backorders bar fill and the flag borders are fixed categorical uses, not judgments).
- Labels ≤ 3 words where possible.
- If a metric VALUE cannot be computed, show "N/A" in the card (a delta slot with no prior data shows a plain hyphen "-" per the Global Delta Rule). Never leave a card blank or skip it.
- **Cap total distinct visual elements at 6 across the report.** The ledger: (1) Net Sales vs. Target composite card, (2) Supply Coverage card, (3) Backorders & Stockouts card, (4) Sell-Through & Turns card, (5) Days on Hand card + Margin/AOV row rendered as one inventory-and-profitability band, (6) Flags card. Merge or cut before rendering if a draft exceeds this; do not add a seventh without removing one.
---
## Data Source Mapping
Tools listed are typical defaults for a physical goods stack; always read from whichever equivalent system is actually connected, and note any substitution in the affected section.
| Metric | Source | Field / Object | Notes |
|--------|--------|----------------|-------|
| Monthly net sales target | ERP (SAP S/4HANA or NetSuite) | Budget/goal record for current month, company-wide, post-returns | If not found, stop and ask the exec before generating |
| Net sales (MTD and weekly) | ERP | SUM(order line units × price) - returns/credit memos, per window | Always post-returns; see Net, Not Gross |
| Units sold / returns $ / returns rate | ERP | Sales order lines; credit memos or RMA records | Returns rate = returns $ ÷ gross sales, no arrow |
| Required $/day to hit monthly target | Derived | (Monthly target - net sales MTD) ÷ days remaining in month | Benchmark for judging MTD pacing |
| Projected month-end at pace | Derived | (Net sales MTD ÷ days elapsed) × days in month | Neutral extrapolation; delta slot always "-" |
| Forecasted demand (current cycle) | Planning tool (Kinaxis RapidResponse or SAP IBP) | Consensus demand plan for current month | Never derived by Claude from sales history |
| Projected supply position | Planning tool | On-hand + confirmed inbound receipts landing within the cycle | |
| Supply coverage % | Derived | Projected supply ÷ forecasted demand × 100 | Judged vs. 100% |
| Sell-through rate (fast movers) | Inventory tool (Cin7 or NetSuite) | Units sold ÷ units available at window start, top SKUs by trailing-month revenue | Weekly window |
| Inventory turns (portfolio) | Inventory tool or ERP | Annualized COGS ÷ average inventory value, calendar month | Monthly rollup line |
| Backorder $ exposure | ERP or inventory tool (SAP S/4HANA or Fishbowl) | SUM(open order line value) WHERE unfulfilled past promise date | |
| Stockout SKU count | Inventory tool | COUNT(SKUs) WHERE available = 0 AND open demand exists | |
| New orders $ this week | ERP | SUM(order value) WHERE created within current 7-day window | |
| Days on hand (current) | Inventory tool (Blue Yonder or Cin7) | On-hand units ÷ average daily units sold, trailing 4 weeks | Judged vs. target band |
| Days on hand (monthly average) | Inventory tool | Average of daily DOH across calendar month | The reported exec figure |
| DOH target band | Planning tool or ERP settings | Min-max days | If unavailable after asking once, render DOH neutral with a note |
| Gross margin % | ERP or BI layer (NetSuite or Power BI) | (Net sales - COGS) ÷ net sales, calendar month | Neutral |
| Average order value | ERP | Net sales ÷ order count, calendar month | Neutral |
| Largest stockout risk | BI exception layer (Power BI or Tableau) if connected, else derived from planning + ERP data | Largest (forecasted units at risk × price) WHERE projected supply < forecasted demand | Single largest by $ sales impact only, not a list |
| Largest fulfillment/supplier delay | ERP or inventory tool | Largest open PO/inbound shipment $ value WHERE expected date < today | Single largest only; include days late |
| PoP delta (all metrics) | Derived | (current period value - prior period value) ÷ prior period value × 100, per that metric's cadence window | Rendered per the Global Delta Rule; color per Judgment Rules |
---
## Report Structure
Render the header block first and the footer last (both specified in the Style Contract). Produce sections in this exact order:
1. **Summary: What's Going On** *(mandatory, always first, bulleted, not prose)*
   Two bullet groups:
   - **"Wins This Week"**: 2-4 bullets. Achievements and positive movement vs. the prior period, with a number in each (e.g. "Net sales hit $412K this week vs. $336K last week (↑22.6%)"). This goes first; the report can't read as only a list of problems.
   - **"Focus For This Week"**: 2-4 bullets. Specific, actionable gaps naming the actual SKU, supplier, or category (e.g. "Supply covers 91% of forecasted demand, below the 100% benchmark" or "Alpine 2-Person Tent at $84K sales impact is the largest stockout risk"). Bullet counts don't need to be even; use however many genuinely apply.
   Each bullet ≤20 words and carries a number. No "significant," "up," or "trending" without a figure. Prior-period numbers may appear in Summary prose for context, but the change itself always renders per the Global Delta Rule in parentheses; never a bare "up 22.6%" without the arrow. If there's genuinely no win to report, state a neutral fact instead of fabricating one (e.g., "Net sales held flat week over week").
2. **Net Sales vs. Target** *(Monthly primary; dominant composite card: MTD number with MoM delta, progress bar showing MTD attainment % vs. 100%, this-week tracking subline with WoW delta, returns note, pacing sub-KPI row with required $/day in judged color and projected month-end neutral with its "-" delta slot; full anatomy in the Style Contract)*
3. **Forecast vs. Supply Position** *(Monthly; single KPI card: coverage % in judged color, benchmark stated inline, math subline showing projected supply and forecasted demand; amber weekly-exception callout line only when triggered)*
4. **Backorders & Stockouts** *(Weekly; backorder $ exposure headline with WoW delta, New Orders vs. Backorders bars, stockout SKU count line)*
5. **Inventory Health** *(one band, two halves: Sell-Through & Turns half with the weekly fast-mover rate + monthly turns rollup line; Days on Hand half with the judged current DOH + band inline + monthly average subline)*
6. **Margin & AOV** *(Monthly; 2-up KPI row: gross margin % and AOV, each with a neutral MoM delta and window note)*
7. **Flags: What Needs a Decision** *(Weekly, point-in-time; small card, two labeled lines: largest stockout risk with $ sales impact and its signal, largest fulfillment/supplier delay with $ value and days late)*
---
## Design Language
- **UI library:** Plain HTML + inline CSS (fully self-contained artifact, no external dependencies)
- **Visual style:** Bloomberg terminal × Metabase dark mode; data-dense, zero decoration
- **Punctuation, hard rule: no em dashes or en dashes anywhere in the rendered report.** Not in titles, subtitles, section headers, labels, captions, table cells, notes, bullets, or chips. Use colons, commas, periods, or the middle dot ( · ) separator instead. Plain hyphens are allowed; the ban is on em and en dashes. Date ranges use a plain hyphen (Jul 7-13). Missing VALUES render as "N/A"; a delta slot with no prior data renders a plain hyphen "-" per the Global Delta Rule.
- **Number formatting:** dollar amounts ending in three zeros abbreviate with K ($280,000 renders as "$280K"); larger round amounts abbreviate with M ($4,200,000 renders as "$4.2M"); non-round amounts render in full. Unit counts follow the same K/M abbreviation. The currency code appears exactly once, in the footer (e.g. "All figures in USD"), determined per the Prerequisites currency rule, never appended to individual numbers.
- **Background:** #0f1117
- **Surface / card background:** #1a1d27
- **Primary text:** #f0f0f0
- **Secondary / label text:** #c4c8d0; must meet **WCAG AA contrast (≥4.5:1)** against whatever it sits on. Verify against both page background and card background before finalizing.
- **Text rendered inside a colored bar/segment** must contrast against *that segment's fill color*, not the page background; typically near-white on saturated fills. Check each fill color individually.
- **Accent color:** #3b82f6 (blue): progress bars, active highlights, links, new-orders bar fill
- **Judged-good:** #22c55e (green), only on metrics listed in the Judgment Rules table
- **Judged-monitor:** #f59e0b (amber), only on metrics listed in the Judgment Rules table, plus the fixed weekly-exception callout line and the supplier-delay flag border
- **Judged-bad:** #ef4444 (red), only on metrics listed in the Judgment Rules table (fixed categorical uses excepted: the backorders bar fill and the stockout-risk flag border)
- **Neutral delta (everything else):** ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0, never green/red
- **Stockout-risk flag:** #ef4444 left border on its line in the Flags card
- **Supplier-delay flag:** #f59e0b left border on its line in the Flags card
- **Font:** one sans-serif stack for everything, including numbers: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif. Numbers are bold with font-variant-numeric: tabular-nums. No monospace, no webfonts, no external fonts.
- **Font size floor: 14px everywhere, no exceptions**, including chart annotations and badges. Nothing in the report may render smaller than 14px.
- **Number size:** 28px for North Star numbers; 18px for accessory metrics; 14px for labels and chart annotations
- **Layout:**
  - Full-width single column
  - KPI cards in a 3-up row max
  - Delta/judgment badge inline, right-aligned, on the metric itself
- **One dominant focal point:** the Net Sales vs. Target MTD progress bar card is the largest element on the page.
- **Chart titles state the finding**, not the variable names.
- **6-visual-element cap** is a hard constraint, not a suggestion.
- **Before shipping, spot-check every text/background pairing used in the report against a contrast checker.** This is a required step, not implied by picking colors from this palette.
---
## Style Contract (reproduce this exact rendering every run)
The report must look identical from run to run, and identical in style to the other report skills in this system. Do not restyle, reinterpret, or "improve" the visual design. Implement exactly:
- **Page:** background #0f1117, content max-width 1100px, 32px page padding, single column, 16px vertical gap between cards.
- **Section eyebrows:** every section opens with an eyebrow line sitting above its card, outside it: 14px, uppercase, letter-spacing 0.08em, #c4c8d0, middle-dot separators. Weekly sections carry the week (e.g. "BACKORDERS & STOCKOUTS · WEEK OF JUL 7-13"); Monthly sections carry the month (e.g. "NET SALES VS. TARGET · JULY 2026", "FORECAST VS. SUPPLY · JULY 2026"). The Inventory Health band carries both windows in one eyebrow: "INVENTORY HEALTH · WEEK OF JUL 7-13 · MONTHLY ROLLUPS: JULY 2026".
- **Header block:** eyebrow "S&OP REVIEW · <month year>"; company/org name 24px bold #f0f0f0; subline 14px #c4c8d0 "Week of <Jul 7-13, 2026> · Generated <date>". Top-right status chip: #1a1d27 pill, 1px border rgba(255,255,255,0.08), border-radius 999px, 8px colored status dot (green/amber/red per the supply coverage judgment) + 14px text (e.g. "Supply covers 104% of forecast").
- **Cards:** #1a1d27 background, 1px solid rgba(255,255,255,0.07) border, border-radius 10px, 20-24px padding. Card-internal section titles: 14px uppercase letterspaced #c4c8d0.
- **Summary card:** 3px #3b82f6 left border; two-column layout; group headers 14px uppercase: "✓ WINS THIS WEEK" in #22c55e, "⚑ FOCUS FOR THIS WEEK" in #f59e0b; bullets 14px #f0f0f0 with dim bullet glyphs.
- **KPI pattern:** label 14px #c4c8d0 above the number; number 28px (North Star) or 18px (accessory) bold #f0f0f0; PoP delta inline immediately after the number, 14px, per the Global Delta Rule.
- **Net Sales card anatomy:** internal title "MTD PROGRESS: <TARGET> MONTHLY TARGET" with the target abbreviated per the number formatting rule. Dominant line "Net Sales MTD" 28px bold #f0f0f0 with its MoM delta inline. Directly beneath, one 14px #c4c8d0 labeled subline: "This week: <value>" with its WoW delta inline. Beneath that, one 14px #c4c8d0 returns note: "Returns MTD: <value> · <x>% of gross" (no arrow on the rate). If returns data is unavailable, replace that note with "Returns data unavailable: figures shown are gross sales." Progress bar and axis below (0 / "Day X of <days in month>" / target). Then a 2-up sub-KPI row: (1) Required $/Day to Hit Target in the judged color, meaning the dollar amount text itself renders in that color, with a green check mark immediately to its right when judged green, and directly below it a line in the same judged color stating actual pace vs. required (e.g. "Actual $14.2K/day · 8% above required" or "below required"); (2) Projected Month-End at Pace with subline "<X>% of target · neutral estimate", its delta slot always a plain hyphen "-", never "N/A".
- **Progress bar:** 8px tall, border-radius 4px, fill #3b82f6, track #262a36; axis line below: left "0", center "Day X of <days in month>", right the target amount abbreviated per the number formatting rule, all 14px #c4c8d0.
- **Supply Coverage card anatomy:** internal title "PROJECTED SUPPLY VS. FORECASTED DEMAND"; dominant number "<XXX>%" 28px bold in the judged color; immediately right of it, 14px #c4c8d0 "benchmark 100%"; directly below, one 14px #c4c8d0 subline: "<projected supply> projected supply · <forecasted demand> forecasted demand". If the weekly exception check triggered, one 14px #f59e0b callout line above the dominant number: "Weekly exception: coverage fell below 100% mid-cycle." MoM delta inline on the coverage % only if a prior-month coverage figure exists, else "-".
- **Backorders & Stockouts card anatomy:** headline "Backorder Exposure: <value>" 28px bold #f0f0f0 with its WoW delta inline. Below it, one bar per category (New Orders, Backorders): New Orders fill #3b82f6, Backorders fill #ef4444, category label left 14px #c4c8d0, value and WoW delta right 14px #c4c8d0. Below the bars, one plain 14px #c4c8d0 line: "<N> SKUs stocked out with open demand" with its WoW delta.
- **Inventory Health band anatomy:** one card, two internal halves separated by a 1px rgba(255,255,255,0.07) vertical hairline. Left half "SELL-THROUGH & TURNS": fast-mover sell-through rate 18px bold #f0f0f0 with its WoW delta and 14px #c4c8d0 note "top SKUs · current week"; below it a 14px #c4c8d0 line "Portfolio turns: <X.X>x annualized" with its MoM delta. Right half "DAYS ON HAND": current DOH 28px bold in the judged color; immediately right, 14px #c4c8d0 "target <min>-<max> days"; below, 14px #c4c8d0 line "Monthly average: <X> days" with its MoM delta. If no band is set, DOH renders #f0f0f0 with note "No target band set: shown without judgment."
- **Margin & AOV row anatomy:** 2-up KPI row: Gross Margin % and Average Order Value, each 18px bold #f0f0f0 with its 14px #c4c8d0 label above, MoM delta inline, and "calendar month" window note below.
- **Flags card anatomy:** two labeled lines. "Largest Stockout Risk": product name, $ sales impact at risk, and the signal (e.g. "projected supply covers 12 of 30 forecast days"), 3px #ef4444 left border on this line. "Largest Fulfillment/Supplier Delay": supplier or PO reference, $ value, and days late, 3px #f59e0b left border on this line. If no SKU qualifies as a stockout risk this week, render "No stockout risks currently flagged." in place of that line. If no PO or shipment is late, render "No fulfillment or supplier delays currently flagged." in place of that line.
- **Pills:** category pills (SKU category, delay type) where used in table or card context: #262a36 background, 4px radius, 14px text.
- **Footer:** hairline top border; left side "<Company/org name> · Week of <range> · <Month Year> · All figures in <currency>"; right side "Data: <connected ERP> · <connected inventory tool> · <connected planning tool><· BI layer if connected> · sales-ops-planning-review v1.0"; both 14px #c4c8d0.
- If a styling question is not answered here, resolve it with the nearest token above. Never introduce a new color, font, size, or component style.
---
## Output Contract
- **File type:** Single self-contained HTML file (all CSS inline, no external scripts or fonts)
- **Delivery method:** Rendered as an artifact in the chat
- **Required sections checklist:**
  - [ ] Header block + status chip (per Style Contract, chip judged on supply coverage)
  - [ ] Summary / What's Going On (mandatory, always first, bulleted: Wins This Week + Focus For This Week)
  - [ ] Net Sales vs. Target (MTD headline + weekly tracking subline + returns note + pacing sub-KPI row)
  - [ ] Forecast vs. Supply Position (judged vs. 100%, benchmark stated inline, weekly-exception callout when triggered)
  - [ ] Backorders & Stockouts (exposure headline + New Orders vs. Backorders bars + stockout SKU line)
  - [ ] Inventory Health (sell-through + turns half, days on hand half judged vs. band)
  - [ ] Margin & AOV (2-up KPI row, neutral MoM deltas)
  - [ ] Flags: What Needs a Decision (stockout risk + supplier delay, single largest each, not a list)
  - [ ] Footer (per Style Contract)
- **Tone:** Operational and exec-shareable; specific numbers and named SKUs/suppliers, achievements included, no fluff. Written for the exec ("your supply position"/"the business"), not first-person ops voice.
- **Punctuation:** zero em dashes and zero en dashes anywhere in the output, including titles, labels, and table cells. Missing values are "N/A"; a delta slot with no prior data to compare is a plain hyphen "-".
- **Reading level and length:** Summary is bulleted, not prose; 4-8 bullets total, ≤20 words each. Cards are not subject to this limit, but no text anywhere renders below the 14px floor.
- **Total visual elements:** ≤6 across the whole report, per the ledger in the Visualization Guide.
- **PoP display:** inline on every metric, formatted per the Global Delta Rule (arrow + one-decimal %), spanning WoW or MoM per that metric's cadence per the Cadence Rule. Neutral #c4c8d0 by default; colored judgment only for the three metrics in the Judgment Rules table.
- **Style:** must match the Style Contract exactly, run to run.
---
## Fallback Rules
| Scenario | Claude should… |
|----------|----------------|
| Monthly net sales target not found in the ERP | Stop. Ask the exec for the monthly target. Do not generate any part of the report until provided. |
| Days-on-hand target band not found | Ask the exec once. If not provided, render DOH neutral #f0f0f0 with the "No target band set" note per the Style Contract. Do not block. |
| Currency mixed or cannot be determined from ERP records | Stop. Ask the exec which currency to report in before generating. |
| Returns data unavailable for the window | Render gross sales with the "Returns data unavailable: figures shown are gross sales." note per the Style Contract. Do not block, never silently substitute. |
| No planning tool connected | Render the Forecast vs. Supply card with "N/A" values and note: "No planning tool connected: forecast and supply position unavailable." Status chip falls back to the net sales pacing judgment. Do not derive a forecast from sales history. |
| No inventory tool connected but ERP carries inventory data | Read stock metrics from the ERP and note the substitution at the top of the affected sections. Do not block. |
| No orders in the current 7-day window | Show weekly figures as $0 / 0 with a neutral delta. Do not skip the section. |
| Prior comparable period (per that metric's cadence) returns no data | Show a plain hyphen "-" in every affected delta slot plus a one-line note: "No prior period data available." Missing VALUES in cells still render "N/A". |
| No genuine "win" to report this week | State a neutral factual bullet in the Summary instead of fabricating a positive spin (e.g., "Net sales held flat week over week"). |
| No SKU qualifies as a stockout risk | Render "No stockout risks currently flagged." in the Flags card. Do not skip the card. |
| No late PO or shipment | Render "No fulfillment or supplier delays currently flagged." in the Flags card. Do not skip the card. |
| Sell-through cannot be computed (no receiving/availability data for the window) | Show "N/A" with note: "Insufficient inventory movement data in the window." |
| Inventory turns cannot be computed (COGS or average inventory unavailable) | Show "N/A" in that line. |
| A metric field is missing on a record | Show "N/A" in that cell. Never infer or fill in missing values. |
| Any connected source returns an error on a query | Note the error at the top of the affected section. Proceed with all other sections; block only on Prerequisites-listed blockers. |
---
## What This Skill Is NOT For
**Fixed, applies to every weekly report skill built from this template:**
- Do not use em dashes or en dashes anywhere in the rendered report: no titles, labels, captions, cells, or notes.
- Do not express a PoP delta in any format other than the Global Delta Rule (arrow + one-decimal %). No "prior period was X", no bare percentages, no arrows without percentages.
- Do not put ↑/↓ arrows on percentages that are not PoP deltas (e.g., supply coverage %, MTD attainment %, sell-through rate, returns rate, gross margin %).
- Do not give an accessory metric its own cadence; it always inherits the cadence of the North Star it explains.
- Do not color-code direction as good/bad on any metric without a listed benchmark in the Judgment Rules table.
- Do not render two bar groups for "this period vs. prior period"; use the single-bar-plus-inline-delta pattern.
- Do not float a derived rate as separate text near a chart; attach it to the element it describes.
- Do not write the Summary as a paragraph; it must be two bulleted groups, wins first.
- Do not present the report as only a list of problems; it must show what's working too.
- Do not fabricate a "win" bullet if there isn't a real one; state a neutral fact instead.
- Do not render any text below the 14px floor.
- Do not exceed the 6-visual-element cap; if content doesn't fit, cut or merge, don't shrink the font.
- Do not deviate from the Style Contract: no new colors, fonts, sizes, or component styles.
- Do not fabricate missing fields; surface the gap instead.
**Role-specific:**
- Do not render a SKU-level inventory table or a full stock listing anywhere in this report; execs get exceptions, not the warehouse ledger. Two flags maximum, one stockout risk and one delay.
- Do not generate demand forecasts or predict future sales; the forecast always comes from the connected planning tool's own consensus plan.
- Do not blend gross and net sales, or silently substitute one for the other; if returns data is missing, disclose it per the Net, Not Gross rule.
- Do not mix cadence windows inside one number or delta slot; a monthly headline may carry one labeled weekly subline, nothing more.
- Do not judge gross margin % or AOV with color; they are intentionally neutral trend metrics in this report.
- Do not create or modify records in any connected system: no purchase orders, no inventory adjustments, no order edits, no forecast overrides.
- Do not rank the stockout-risk flag by SKU count or strategic importance; the flag is chosen by $ sales impact only.
- Do not render per-warehouse, per-channel, or per-region breakdowns unless explicitly asked; this report is company-wide.
- Do not compare against periods other than those defined by the Cadence Rule unless explicitly asked.
---
## Example Trigger
> "Can you pull the S&OP review? Planning meeting is Monday and I want to know if supply covers the forecast."
> "Run my S&OP review."
> "How's inventory looking this week?"
> "Are we at risk of stockouts anywhere?"
