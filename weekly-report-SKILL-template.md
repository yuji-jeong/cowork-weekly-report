---
# YAML frontmatter — defines how Co-work identifies and routes this skill
name: [skill-name]  # e.g., weekly-pipeline-report, deal-health-snapshot
description: [One sentence — what this report does and when to trigger it. e.g., "Generates a weekly pipeline health report from HubSpot and Fireflies data; trigger when reviewing deals at the start of each week."]
version: 1.0
author: [Your Name]
data_sources:
  - [MCP name]  # e.g., HubSpot
  - [MCP name]  # e.g., Fireflies
  - [MCP name]  # e.g., Google Sheets — add or remove rows as needed
trigger_phrases:
  - "[natural language phrase]"  # e.g., "run my pipeline report"
  - "[natural language phrase]"  # e.g., "give me the weekly deal summary"
  - "[natural language phrase]"  # e.g., "how's my pipeline looking"
output_format: [e.g., HTML artifact, inline markdown, PDF export]
---

<!-- ============================================================
     REPORT SKILL TEMPLATE
     Fill in every [BRACKET] with your own values.
     Delete these comments when you're done.
     ============================================================ -->

# [Skill Display Name]

<!-- Goal: Answers "why does this report exist?" — forces you to build around a decision, not just data. -->
## Goal

[One paragraph describing the decision or action this report enables.
Example: "This report helps AEs decide which deals to push this week and which to let sit. It surfaces stalled deals, upcoming close dates, and any accounts that went dark after a demo — so the rep can prioritize outreach without digging through HubSpot manually."]

---

<!-- North Star Metric: The one number the whole report is built to explain. If you can't name it, the report will feel unfocused. -->
## North Star Metric

**Metric name:** [e.g., Weighted Pipeline Value]

**Calculation:** [e.g., SUM(deal_amount × probability) for all open deals in current quarter]

**Why it matters:** [e.g., This is the number your manager checks. If it drops, you need a story — this report gives you that story before the 1:1.]

---

<!-- Accessory Metrics: The "why" behind the North Star. Each one explains a slice of movement. Keep it to 3–5 or the report becomes noise. -->
## Accessory Metrics

### 1. [Metric Name]
- **Calculation:** [How it is calculated — formula or plain English]
- **What it reveals:** [What business behavior this exposes]

### 2. [Metric Name]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]

### 3. [Metric Name]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]

### 4. [Metric Name — optional]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]

### 5. [Metric Name — optional]
- **Calculation:** [How it is calculated]
- **What it reveals:** [What business behavior this exposes]

---

<!-- Data Source Mapping: Tells Claude exactly where to pull each number. The more specific, the fewer hallucinations. -->
## Data Source Mapping

| Metric | Source | Field / Object | Notes |
|--------|--------|----------------|-------|
| [Metric name] | [MCP name] | [e.g., deals.amount, contacts.last_activity_date] | [e.g., filter to stage = "Demo Done"] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |
| [Metric name] | [MCP name] | [Field / Object] | [Notes] |

---

<!-- Report Structure: The exact layout Claude must produce. Calling out format (table vs narrative vs card) prevents Claude from making style decisions for you. -->
## Report Structure

Produce sections in this exact order:

1. **[Section name]** — [format: e.g., KPI card row] — [what it shows]
2. **[Section name]** — [format: e.g., table, sorted by close date] — [what it shows]
3. **[Section name]** — [format: e.g., 2–3 sentence narrative] — [what it shows]
4. **[Section name]** — [format: e.g., bulleted list] — [what it shows]
5. **[Section name]** — [format: e.g., chart or sparkline] — [what it shows]

---

<!-- Design Language: Locks in the visual style so the output looks intentional, not like default ChatGPT output. -->
## Design Language

- **UI library:** [e.g., shadcn/ui, Radix, Tailwind CSS, Ant Design, plain HTML + inline styles]
- **Visual style keyword:** [e.g., liquid glass, bento grid, editorial dark, clean white card, warm neutral]
- **Key layout notes:**
  - [e.g., Use a two-column bento grid for KPI cards]
  - [e.g., Keep font sizes ≥14px for legibility in Slack screenshots]
  - [e.g., Use a dark background (#0f0f0f) with white text]
  - [e.g., Accent color: #[HEX]]

---

<!-- Output Contract: Defines "done." Without this, Claude stops at different points each time. -->
## Output Contract

- **File type:** [e.g., single self-contained HTML file, inline markdown block]
- **Delivery method:** [e.g., rendered as an artifact in the chat, pasted inline, saved to Google Drive]
- **Required sections checklist:**
  - [ ] [Section 1]
  - [ ] [Section 2]
  - [ ] [Section 3]
  - [ ] [Section 4]
  - [ ] [Section 5]
- **Tone:** [e.g., executive summary — punchy, no fluff; operational — specific numbers and actions; self-review — honest, first-person]
- **Reading level and length:** [e.g., 8th grade, under 400 words of narrative; tables and cards can be longer]

---

<!-- Fallback Rules: What happens when data is missing. This prevents Claude from making up numbers or silently skipping sections. -->
## Fallback Rules

| Scenario | Claude should… |
|----------|----------------|
| A data source returns no results | [e.g., Insert a clearly labeled "No data available" placeholder — do not skip the section] |
| A metric cannot be computed | [e.g., Show the metric card with "—" and a one-line note explaining why] |
| A connected MCP is unavailable | [e.g., Note the outage at the top of the report and proceed with available sources] |
| [Add your own scenario] | [What Claude should do] |

---

<!-- What This Skill Is NOT For: Draws a hard boundary. Prevents scope creep and hallucinated analysis. -->
## What This Skill Is NOT For

- [e.g., Do not generate forecast models or predict close probability — use the CRM's own probability field]
- [e.g., Do not summarize email threads or call recordings unless explicitly asked]
- [e.g., Do not create new CRM records or update deal stages]
- [e.g., Do not compare this period against prior periods unless historical data is explicitly provided]
- [e.g., Do not fabricate missing fields — surface the gap instead]

---

<!-- Example Trigger: Shows the user exactly how to activate this skill naturally. Makes onboarding frictionless. -->
## Example Trigger

> "[Realistic plain-language example of how a user would invoke this skill in Co-work. e.g., 'Hey, can you pull my pipeline report? I have a 1:1 in 30 minutes and want to know which deals I should flag.']"
