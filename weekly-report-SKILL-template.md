---
# YAML frontmatter — defines how Co-work identifies and routes this skill
name: [skill-name]  # e.g., weekly-ops-report, team-performance-snapshot
description: [One sentence — what this report does and when to trigger it. e.g., "Generates a weekly operations health report from Zendesk and Google Sheets data; trigger when reviewing ops metrics at the start of each week."]
version: 3.1
author: yuji-jeong
data_sources:
  - [MCP name]  # e.g., Zendesk
  - [MCP name]  # e.g., Google Sheets — add or remove rows as needed
trigger_phrases:
  - "[natural language phrase]"  # e.g., "run my ops report"
  - "[natural language phrase]"  # e.g., "give me the weekly ops summary"
  - "[natural language phrase]"  # e.g., "how's the queue looking"
output_format: HTML artifact
---

<!-- ============================================================
     WEEKLY REPORT SKILL TEMPLATE
     Fill in every [BRACKET] with your role and metrics.
     Everything NOT in brackets is a fixed design/hard rule —
     carried over as-is so every weekly report skill built from
     this template looks and behaves like the same system.
     Delete comments when you're done.
     ============================================================ -->

# [Skill Display Name]

<!-- Goal: Answers "why does this report exist?" — forces you to build around a decision, not just data. -->
## Goal

[One paragraph describing the decision or action this report enables. It is a **week-over-week directional tool**, not a statistically validated forecast; small sample sizes are expected and fine. It is also written to be **shareable with a manager as-is**: it must show what's going well, not just what's broken. Target: a 60-second read that ends with the reader knowing what to do next.
Example: "This report helps ops leads decide which queues need attention this week and which are healthy. It surfaces SLA breaches, backlog growth, and any process that degraded after a change, so the lead can prioritize without digging through the ticketing tool manually."]

---

<!-- Prerequisites: What must be connected/available before the report can run, and what blocks generation entirely if missing. -->
## Prerequisites

- **[Connector name] connected**: [what data comes from here, e.g., "ticket records, SLA timestamps, and queue volume"].
- **[Connector name] connected**: [what data comes from here].
- **[Any hard-required number, e.g., a target/quota/SLA threshold] set in [system]**: the report cannot generate without this number. If it cannot be found, Claude must ask the user to provide it before proceeding. Do not generate the report without it.

---

<!-- Reporting Cadence: Declares which period(s) this report runs on. A report can be single-cadence (the common case) or mix cadences across different North Stars (e.g., a weekly activity North Star alongside a quarterly OKR North Star in the same report). Every other section's period language (deltas, section eyebrows, header/footer) resolves from whatever is declared here. -->
## Reporting Cadence

**Cadence(s) in this report:** [Weekly / Monthly / Quarterly / Yearly — list one or more]

| Cadence | Comparison window | Label format |
|---------|-------------------|---------------|
| Weekly | Trailing 7 days vs. prior 7 days (WoW) | "Week of Jul 7-13" |
| Monthly | Calendar month vs. prior calendar month (MoM) | "July 2026" |
| Quarterly | Calendar quarter vs. prior quarter (QoQ) | "Q3 2026" |
| Yearly | Fiscal/calendar year vs. prior year (YoY) | "FY2026" |

[Delete rows for cadences this report doesn't use. If this report uses only one cadence, every metric and section below inherits it automatically — you don't need to tag anything per-metric.]

---

<!-- North Star Metric(s): The number(s) the whole report is built to explain. If you can't name it, the report will feel unfocused.
     Use Option A for a single-question report. Use Option B (one metric per decision) when the report needs to answer several distinct questions — this scales better once a report grows past 2-3 questions.
     Cadence: only declare a Cadence per North Star if this report mixes cadences (per Reporting Cadence above). If the whole report runs on one cadence, skip the Cadence field/column entirely — it's implied. -->
## North Star Metric(s)

**Option A — single North Star:**
**Metric name:** [e.g., Weighted Pipeline Value]
**Cadence:** [Only if this report mixes cadences — e.g., Quarterly. Otherwise omit; it's whatever's declared in Reporting Cadence.]
**Calculation:** [e.g., SUM(deal_amount × probability) for all open deals in current quarter]
**Why it matters:** [e.g., This is the number your manager checks. If it drops, you need a story — this report gives you that story before the 1:1.]

**Option B — one North Star per decision (use when the report must answer multiple questions):**
| # | Decision | North Star Metric | Cadence *(omit column if single-cadence)* |
|---|----------|-------------------|---------------------------------------------|
| 1 | [e.g., Is the queue healthy?] | [e.g., Backlog size + tickets aged over SLA] | [e.g., Weekly] |
| 2 | [e.g., Am I responding fast enough?] | [e.g., First response time vs. SLA target] | [e.g., Weekly] |
| 3 | [Add rows as needed, one per real decision] | [North Star metric for that decision] | [Cadence] |
| — | Did I move up or down vs. the prior period? | Period-over-period delta on every metric above, per its own cadence (inline, on the metric itself, not a separate section) | — |

---

<!-- Accessory Metrics: The "why" behind the North Star. Each one explains a slice of movement. Keep it to 3–5 per North Star or the report becomes noise.
     Cadence: accessory metrics do NOT get their own cadence. Each one always runs on the same cadence as the North Star it explains — that's its whole purpose, so there's nothing to declare separately. If Option B produced North Stars on different cadences, use "Explains" to say which one this accessory metric belongs to and its cadence follows from that automatically. -->
## Accessory Metrics

### 1. [Metric Name]
- **Explains:** [Which North Star this supports — required only if you used Option B with multiple North Stars; its cadence follows automatically from that North Star]
- **Calculation:** [How it is calculated — formula or plain English]
- **What it reveals:** [What business behavior this exposes]
- **Delta type:** [Neutral (no benchmark exists) / Judged (a real benchmark exists — add to the Judgment Rules table below)]

### 2. [Metric Name]
- **Explains:** [North Star it supports, if applicable]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]
- **Delta type:** [Neutral / Judged]

### 3. [Metric Name]
- **Explains:** [North Star it supports, if applicable]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]
- **Delta type:** [Neutral / Judged]

### 4. [Metric Name — optional]
- **Explains:** [North Star it supports, if applicable]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]
- **Delta type:** [Neutral / Judged]

### 5. [Metric Name — optional]
- **Explains:** [North Star it supports, if applicable]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]
- **Delta type:** [Neutral / Judged]

---

<!-- Global Delta Rule: FIXED. This is the single, universal format for period-over-period change in every report skill built from this template. Do not alter it per role — it's what makes deltas scannable across every report you build. The comparison window itself (WoW/MoM/QoQ/YoY) resolves from Reporting Cadence, per metric; the format below never changes regardless of which window applies. -->
## Global Delta Rule (applies to every number in the report)

This is the single, universal format for period-over-period (PoP) change. It applies to every metric, card, chart, and table cell. No exceptions.
- A PoP delta is always and only rendered as an arrow followed by the percentage to one decimal place: ↑16.7% or ↓4.2%. The period it spans (WoW, MoM, QoQ, YoY) is whatever that metric's Cadence resolves to per Reporting Cadence, but the rendered format is identical either way.
- Never "prior period was X". Never a bare percentage without an arrow. Never an arrow without a percentage. Never "flat" or "=" in any delta slot.
- If the delta rounds to 0.0%, render ↑0.0% (zero or positive gets ↑, negative gets ↓).
- If a PoP delta cannot be computed because there is no prior period to compare, render a plain hyphen "-" in the delta slot. The hyphen means "no previous period to compare". Never "N/A" in a delta slot; "N/A" stays the indicator for missing VALUES in cells and cards.
- No suffix text on the delta itself (no "vs prior period", no "WoW"/"MoM"/etc.). Section eyebrows already carry the period context.
- ↑ and ↓ are reserved strictly for PoP deltas. A percentage that is not a PoP delta (e.g., a conversion rate, an attainment %, a source/category split) never gets an arrow.
- Delta color: neutral gray (see Design Language) by default. Green/amber/red only on metrics listed in the Judgment Rules table, and even then the format stays arrow + one-decimal percentage.

---

<!-- Judgment Rules: FIXED structure. Direction is a fact by default; color is only earned when a real benchmark exists. Fill in which of YOUR metrics are judged and against what benchmark. -->
## Judgment Rules: When Color Means an Opinion, and When It's Just a Fact

**The default: direction is a fact, not an opinion.** Whether a number went up or down is objectively true. Whether "up" is *good* usually depends on context the report doesn't have. So by default, direction is neutral.
- **Neutral metrics (no benchmark exists):** delta rendered per the Global Delta Rule in neutral gray next to the number. No green, no red. This is every metric NOT listed in the table below.
- **Judged metrics (a real benchmark exists):** the only places allowed to render an opinion, because there's a defensible number to compare against.

| Metric | Benchmark | Green (good) | Amber (monitor) | Red (bad) |
|---|---|---|---|---|
| [e.g., SLA compliance rate] | [e.g., Trailing 8-week average] | [e.g., At or above benchmark] | [e.g., Within 10 pts below] | [e.g., More than 10 pts below] |
| [Metric] | [Benchmark] | [Threshold] | [Threshold] | [Threshold] |
| [Metric] | [Benchmark] | [Threshold] | [Threshold] | [Threshold] |

The benchmark's trailing window should match that metric's own cadence, not default to weekly: a weekly metric benchmarks against a trailing-weeks average, a quarterly metric against a trailing-quarters average, and so on.
If a benchmark can't be computed (see Fallback Rules), fall back to a neutral delta. Never guess a color.

---

<!-- Visualization Guide: Tells Claude which chart to use for each metric, and the fixed chart rules every weekly report skill obeys. Pick the chart by the MESSAGE you want to send, not by the data type. -->
## Visualization Guide

Match each metric to the message you want it to send, then use the chart for that message. Do not let Claude pick charts freely.

| Metric | Message type | Chart to use | Why this chart |
|--------|-------------|--------------|----------------|
| [Metric name] | [Comparison / Trend over time / Part-of-whole / Distribution / Relationship] | [Chart] | [One line] |
| [Metric name] | [Message type] | [Chart] | [One line] |
| [Metric name] | [Message type] | [Chart] | [One line] |

**Message-type → chart cheat sheet:**
- **Comparison** (contrast values across categories) → **bar chart**
- **Trend over time** (change across weeks/months) → **line chart**
- **Part-of-whole** (composition / breakdown) → **stacked bar** (use a pie only for 2–3 slices) or compact inline pills if the breakdown fits in one KPI row
- **Distribution** (spread, clustering, outliers) → **histogram** or **box plot**
- **Relationship** (correlation between two values) → **scatter plot**
- If a metric is a single number, use a KPI card, not a chart.

**Chart rules (fixed — apply to every weekly report skill):**
- Every chart title states the *finding*, not the variable names (e.g., "Backlog grew 12% after the release" — not "Backlog by week").
- **No duplicate bar groups for "this week vs. prior week."** Use one bar per category plus a marker tick for the prior value, labeled at the tick, and an inline delta per the Global Delta Rule.
- **Merge charts that share an axis** before adding a new one (e.g., a category breakdown and its conversion/completion rate can stack in one card instead of two separate charts).
- **Any derived or secondary number (like a rate or a percentage) must be attached directly to the element it describes** — printed on or inside the bar/column/segment it belongs to, never floated as separate text nearby.
- One dominant series per chart; de-emphasize everything secondary (lighter color / thinner line).
- Color is either: (a) neutral gray delta-only, or (b) green/amber/red per the Judgment Rules table. Never green/red on a metric without a listed benchmark.
- Labels ≤ 3 words where possible.
- If a metric VALUE cannot be computed, show "N/A" in the card (a delta slot with no prior data shows a plain hyphen "-"). Never leave a card blank or skip it.
- **Cap total distinct visual elements at [8, or your own number — pick one and hold to it].** Merge or cut before rendering if a draft exceeds this.

---

<!-- Data Source Mapping: Tells Claude exactly where to pull each number. The more specific, the fewer hallucinations. -->
## Data Source Mapping

| Metric | Source | Field / Object | Notes |
|--------|--------|----------------|-------|
| [Metric name] | [MCP name] | [e.g., tickets.status, tickets.created_date] | [e.g., filter to queue = "Support"] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |
| PoP delta (all metrics) | Derived | (current period value - prior period value) ÷ prior period value × 100, where "period" is that metric's Cadence window (7-day, calendar month, quarter, or year) | Rendered per the Global Delta Rule; color per Judgment Rules |

---

<!-- Report Structure: The exact layout Claude must produce. Calling out format (table vs narrative vs card) prevents Claude from making style decisions for you. -->
## Report Structure

Render the header block first and the footer last (both specified in the Style Contract). Produce sections in this exact order:

1. **Summary: What's Going On** *(mandatory, always first, bulleted, not prose)*
   Two bullet groups:
   - **"Wins This [Period]"** (e.g., "Wins This Week", "Wins This Quarter" — match the report's primary cadence label): 2-4 bullets. Achievements and positive movement vs. the prior period, with a number in each. This goes first because the report is shareable with a manager; it can't read as only a list of problems.
   - **"Focus For Next [Period]"**: 2-4 bullets. Specific, actionable gaps. Bullet counts don't need to be even; use however many genuinely apply.
   Each bullet ≤20 words and carries a number. No "significant," "up," or "trending" without a figure. Prior-period numbers may appear in Summary prose for context, but the change itself always renders per the Global Delta Rule in parentheses (e.g., "16 tickets closed vs. 12 last week (↑33.3%)"). If there's genuinely no win to report, state a neutral fact instead of fabricating one.
2. **[Section name]** — [format: e.g., KPI card row] — [what it shows]
3. **[Section name]** — [format: e.g., table, sorted by X] — [what it shows]
4. **[Section name]** — [format: e.g., bulleted list, operational not a chart] — [what it shows]
5. **[Section name]** — [format: e.g., chart or composite visual] — [what it shows]
6. **Period-over-Period Deltas** *(inline layer, not a standalone section — applies the Global Delta Rule and the neutral/judged split to every metric above, displayed inline on the metric itself, per each metric's own cadence)*

---

<!-- Design Language: FIXED. This is the exact visual system every weekly report skill built from this template renders in — carried over from the proven design, not re-decided per role. Only touch the bracketed accent/brand line if you deliberately want a different brand color. -->
## Design Language

- **UI library:** Plain HTML + inline CSS (fully self-contained artifact, no external dependencies)
- **Visual style:** Bloomberg terminal × Metabase dark mode; data-dense, zero decoration
- **Punctuation, hard rule: no em dashes or en dashes anywhere in the rendered report.** Not in titles, subtitles, section headers, labels, captions, table cells, notes, bullets, or chips. Use colons, commas, periods, or the middle dot ( · ) separator instead. Plain hyphens are allowed. Date ranges use a plain hyphen (e.g., Jul 7-13). Missing VALUES render as "N/A"; a PoP delta slot with no prior period to compare renders a plain hyphen "-".
- **Number formatting:** dollar/count amounts ending in three zeros abbreviate with K (e.g., $280,000 renders as "$280K"); non-round amounts render in full. Any unit or currency code appears exactly once, in the footer (e.g., "All figures in CAD"), never appended to individual numbers.
- **Background:** #0f1117
- **Surface / card background:** #1a1d27
- **Primary text:** #f0f0f0
- **Secondary / label text:** #c4c8d0; must meet **WCAG AA contrast (≥4.5:1)** against whatever it sits on. Verify against both page background and card background before finalizing.
- **Text rendered inside a colored bar/segment** must contrast against *that segment's fill color*, not the page background; typically near-white on saturated fills. Check each fill color individually.
- **Accent color:** #3b82f6 (blue) — progress bars, active highlights, links, chart fills. [Replace with your brand accent only if intentional; keep the rest of the palette fixed.]
- **Judged-good:** #22c55e (green), only on metrics listed in the Judgment Rules table
- **Judged-monitor:** #f59e0b (amber), only on metrics listed in the Judgment Rules table
- **Judged-bad:** #ef4444 (red), only on metrics listed in the Judgment Rules table (fixed categorical uses, like a "lost/failed" bar fill or an aged-item flag, are excepted)
- **Neutral delta (everything else):** ↑/↓ + one-decimal % per the Global Delta Rule, #c4c8d0, never green/red, regardless of which cadence's comparison window it spans
- **Font:** one sans-serif stack for everything, including numbers: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif. Numbers are bold with font-variant-numeric: tabular-nums. No monospace, no webfonts, no external fonts.
- **Font size floor: 14px everywhere, no exceptions**, including chart annotations, badges, and any text inside a chart element.
- **Number size:** 28px for North Star numbers; 18px for accessory metrics; 14px for labels and chart annotations
- **Layout:**
  - Full-width single column
  - KPI cards in a 3-up row max
  - Tables full-width with zebra striping (#1a1d27 / #1f2333)
  - Delta/judgment badge inline, right-aligned, on the metric itself
- **One dominant focal point:** the North Star metric's KPI card or progress element is the largest thing on the page.
- **Chart titles state the finding**, not the variable names.
- **Visual-element cap** (set in the Visualization Guide) is a hard constraint, not a suggestion.
- **Before shipping, spot-check every text/background pairing used in the report against a contrast checker.** This is a required step, not implied by picking colors from this palette.

---

<!-- Style Contract: FIXED structure, carried over so every report from this template is reproducible run to run. Fill in the bracketed card/section names only — do not restyle the anatomy itself. -->
## Style Contract (reproduce this exact rendering every run)

The report must look identical from run to run. Do not restyle, reinterpret, or "improve" the visual design. Implement exactly:
- **Page:** background #0f1117, content max-width 1100px, 32px page padding, single column, 16px vertical gap between cards.
- **Section eyebrows:** every section opens with an eyebrow line sitting above its card, outside it: 14px, uppercase, letter-spacing 0.08em, #c4c8d0, middle-dot separators, using that section's own cadence label per the Reporting Cadence table (e.g., "[SECTION NAME] · WEEK OF JUL 7-13" for a weekly section, "[SECTION NAME] · Q3 2026" for a quarterly one). If a report mixes cadences, each section's eyebrow shows its own period, not one report-wide date.
- **Header block:** eyebrow "[CADENCE] [ROLE] REPORT · <period>" (e.g., "WEEKLY OPS REPORT · Q3 2026", using the report's primary/dominant cadence if mixed); name/title 24px bold #f0f0f0; subline 14px #c4c8d0 "<period label per Reporting Cadence> · Generated <date>" (e.g., "Week of Jul 7-13, 2026 · Generated Jul 13, 2026"). Top-right status chip: #1a1d27 pill, 1px border rgba(255,255,255,0.08), border-radius 999px, 8px colored status dot (green/amber/red per the primary judged metric) + 14px text (e.g., "[status phrase]").
- **Cards:** #1a1d27 background, 1px solid rgba(255,255,255,0.07) border, border-radius 10px, 20-24px padding. Card-internal section titles: 14px uppercase letterspaced #c4c8d0.
- **Summary card:** 3px #3b82f6 left border; two-column layout; group headers 14px uppercase: "✓ WINS THIS [PERIOD]" in #22c55e, "⚑ FOCUS FOR NEXT [PERIOD]" in #f59e0b (period label matches the report's primary cadence); bullets 14px #f0f0f0 with dim bullet glyphs.
- **KPI pattern:** label 14px #c4c8d0 above the number; number 28px (North Star) or 18px (accessory) bold #f0f0f0; PoP delta inline immediately after the number, 14px, per the Global Delta Rule.
- **[North Star card anatomy]:** [describe your primary card layout — e.g., dominant number + companion %, progress bar + axis, sub-KPI row below. Model this on a progress-bar-plus-sub-KPI-row pattern if the North Star is a target/attainment metric.]
- **Pills:** category/type pills in tables or KPI rows: #262a36 background, 4px radius, 14px text.
- **Flag/threshold cells** (e.g., an aged or overdue indicator): small bordered box; [define your thresholds and colors, e.g., over X: red text/border on translucent red; X-Y: amber].
- **Tables:** full-width; column headers 14px uppercase letterspaced #c4c8d0; zebra rows #1a1d27 / #1f2333; 12-14px cell padding; flagged rows get a 3px colored left border matching the flag color.
- **Footer:** hairline top border; left side "[name/role] · <period label per Reporting Cadence, e.g., Week of Jul 7-13> · [unit note, e.g., All figures in CAD]"; right side "Data: [data sources] · [skill-name] v[version]"; both 14px #c4c8d0.
- If a styling question is not answered here, resolve it with the nearest token above. Never introduce a new color, font, size, or component style.

---

<!-- Output Contract: Defines "done." Without this, Claude stops at different points each time. -->
## Output Contract

- **File type:** Single self-contained HTML file (all CSS inline, no external scripts or fonts)
- **Delivery method:** Rendered as an artifact in the chat
- **Required sections checklist:**
  - [ ] Header block + status chip (per Style Contract)
  - [ ] Summary / What's Going On (mandatory, always first, bulleted: Wins This Week + Focus For Next Week)
  - [ ] [Section 2]
  - [ ] [Section 3]
  - [ ] [Section 4]
  - [ ] [Section 5]
  - [ ] Footer (per Style Contract)
- **Tone:** Operational and shareable with a manager; specific numbers and actions, achievements included, no fluff. First-person where natural.
- **Punctuation:** zero em dashes and zero en dashes anywhere in the output, including titles, labels, and table cells. Missing values are "N/A"; a delta slot with no prior data to compare is a plain hyphen "-".
- **Reading level and length:** Summary is bulleted, not prose; 4-8 bullets total, ≤20 words each. Tables and cards are not subject to this limit, but no text anywhere renders below the 14px floor.
- **Total visual elements:** ≤[your cap from the Visualization Guide] across the whole report.
- **PoP display:** inline on every metric, formatted per the Global Delta Rule (arrow + one-decimal %), spanning whatever comparison window that metric's Cadence resolves to. Neutral gray by default; colored judgment only for metrics in the Judgment Rules table.
- **Style:** must match the Style Contract exactly, run to run.

---

<!-- Fallback Rules: What happens when data is missing. This prevents Claude from making up numbers or silently skipping sections. -->
## Fallback Rules

| Scenario | Claude should… |
|----------|----------------|
| [Your required blocking metric] not found | Stop. Ask the user for it. Do not generate any part of the report until provided. |
| A data source returns no results for a section | Show the KPI card as 0 / N/A with a neutral delta. Do not skip the section. |
| Prior comparable period (per that metric's Cadence) returns no data | Show a plain hyphen "-" in every affected delta slot plus a one-line note: "No prior period data available." Missing VALUES in cells still render "N/A". |
| A benchmark (trailing average) cannot be computed | Fall back to a neutral gray delta for that metric. Do not guess a color. |
| No genuine "win" to report this week | State a neutral factual bullet instead of fabricating a positive spin. |
| [A specific connector] unavailable | Note it at the top of the affected section, or top of Summary if it's supplementary. Proceed with all other sections; do not block report generation unless it's a Prerequisites-listed blocker. |
| A metric field is missing on a record | Show "N/A" in that cell. Never infer or fill in missing values. |
| A data source returns an error on any query | Note the error at the top of the affected section. Proceed with all other sections. |
| [Add your own scenario] | [What Claude should do] |

---

<!-- What This Skill Is NOT For: Draws a hard boundary. Prevents scope creep and hallucinated analysis. The first block is fixed (applies to every weekly report skill); add role-specific boundaries below it. -->
## What This Skill Is NOT For

**Fixed, applies to every weekly report skill built from this template:**
- Do not use em dashes or en dashes anywhere in the rendered report.
- Do not express a period-over-period delta in any format other than the Global Delta Rule (arrow + one-decimal %). No "prior period was X", no bare percentages, no arrows without percentages.
- Do not put ↑/↓ arrows on percentages that are not PoP deltas.
- Do not give an accessory metric its own cadence; it always inherits the cadence of the North Star it explains.
- Do not color-code direction as good/bad on any metric without a listed benchmark in the Judgment Rules table.
- Do not render two bar groups for "this week vs. prior week"; use the single-bar-plus-marker pattern.
- Do not float a derived rate as separate text near a chart; attach it to the element it describes.
- Do not write the Summary as a paragraph; it must be two bulleted groups, wins first.
- Do not present the report as only a list of problems; it must show what's working too.
- Do not fabricate a "win" bullet if there isn't a real one; state a neutral fact instead.
- Do not render any text below the 14px floor.
- Do not exceed the visual-element cap; if content doesn't fit, cut or merge, don't shrink the font.
- Do not deviate from the Style Contract: no new colors, fonts, sizes, or component styles.
- Do not fabricate missing fields; surface the gap instead.

**Role-specific, fill in:**
- [e.g., Do not generate forecast models or predict outcomes — use the source system's own probability/estimate field]
- [e.g., Do not summarize communications or transcripts beyond what's explicitly specified]
- [e.g., Do not create or modify records in the source system]
- [e.g., Do not compare against periods other than each metric's own current and prior Cadence window unless explicitly asked]
- [e.g., Do not include data outside the logged-in user's own scope/ownership]

---

<!-- Example Trigger: Shows the user exactly how to activate this skill naturally. Makes onboarding frictionless. -->
## Example Trigger

> "[Realistic plain-language example of how a user would invoke this skill. e.g., 'Hey, can you pull my ops report? I have my team sync in an hour and want to know which queues need attention.']"
</content>
