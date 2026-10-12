# The performance update that writes itself

An interactive prototype of a monthly performance dashboard that drafts its own executive summary. It was built as a submission for the Student Assistant, AI & Business Analytics role at Cerivo.

**Live demo:** https://johnyzag.github.io/self-writing-performance-update/
**File:** `index.html` (single file, no build step, opens in any modern browser)

> All company names, brands and figures are fictional. "Meridian Group" and its four brands are invented to resemble a merged GRC software group. They do not represent Cerivo or any of its legacy companies.

---

## 1. The problem

Monthly performance reporting in a multi-brand scale-up usually takes analysts several days. They consolidate data from separate systems, compute variances to plan, and write the narrative a CFO or CEO actually reads. After a merger the problem compounds, because each legacy brand brings its own billing, CRM and KPI definitions.

The narrative is the most time-consuming part and the least standardised. It is also the part that leadership relies on most.

## 2. What the prototype does

| Component | Purpose |
|---|---|
| Auto-drafted summary | A board-style memo generated from the data: headline, drivers of variance, retention, run-rate outlook |
| Needs attention rail | Rule-based flags ranked by severity (off plan, watch, notable upside) |
| KPI band | Revenue vs plan, ARR, customers and logo churn, each with a status and trend |
| Trend chart | MRR actual vs plan for the group or any brand, with the shortfall shaded |
| Variance chart | Diverging bars showing each brand's deviation from plan, with threshold lines |
| Deviation table | Full traffic-light view per brand: revenue, YTD, ARR, customers, net adds, churn |
| Approval workflow | Drafts start as "Draft for review" and require explicit approval before distribution |

## 3. How to use it

1. Open the file or the live link.
2. Use the month selector (top right) to switch reporting periods. The entire summary rewrites itself for the selected month.
3. Hover any underlined figure in the summary, or any flag in the right rail. The source row in the table and the matching bar are highlighted.
4. Use the brand toggle on the trend chart to move from group view to a single brand.
5. **Regenerate draft** rebuilds the narrative, **Copy text** exports it as plain text, and **Approve for distribution** changes its status.
6. The sun icon toggles light and dark themes.

## 4. Design principles

### 4.1 Facts first, narrative second

The summary is not hard-coded text. Generation follows a fixed pipeline:

```
Source data  ->  Derived metrics  ->  Status rules  ->  Reconciliation checks  ->  Narrative assembly  ->  Human approval
(brand x month)   (variance, YTD,     (on plan /        (totals reconcile,        (only from computed    (draft -> approved)
                   streaks, trends,    watch /           no missing months,        facts)
                   run-rate)           off plan)         customer roll-forward)
```

The narrative layer can only describe facts that were computed and checked upstream. This is the main safeguard against an AI-written report stating numbers that do not exist in the data.

### 4.2 Every claim is traceable

Each figure in the text links to the cell or chart element it came from. For a governance, risk and compliance company, the ability to audit an automated draft matters as much as the draft itself.

### 4.3 Facts are separated from hypotheses

The system reports *what* happened and *where*. It does not claim to know *why*. Possible causes are framed as "Suggested questions for brand owners", and the year-end projection is explicitly labelled a run-rate view, not a forecast.

### 4.4 A human stays in the loop

No draft is distributable without approval. This reflects a realistic deployment for board-level reporting and is consistent with the human-oversight principles in AI governance frameworks such as the EU AI Act.

## 5. Business rules

| Metric | On plan | Watch | Off plan |
|---|---|---|---|
| Revenue vs plan | at or above −1% | −1% to −3% | below −3% |
| Customers vs plan | at or above −1% | −1% to −3% | below −3% |
| Logo churn vs target | at or below target | up to +0.3 pp above | more than +0.3 pp above |

A brand's overall status is the worst of its three metric statuses.

**Narrative logic (selected rules):**

1. **Headline.** Group revenue vs plan for the month and YTD, plus ARR growth since January.
2. **Drivers.** The largest negative contributor in absolute EUR terms is named first. A consecutive-miss streak is mentioned if it reaches three months or more, and a widening gap is reported if the variance worsened over two months.
3. **Root-cause signal.** If new-logo intake is below plan, the text points to new business rather than churn as the larger driver.
4. **Offset.** The largest positive contributor is mentioned if it is more than 1% ahead of plan.
5. **Retention.** The brand with the largest churn breach is identified, compared with its target and with its own level three months earlier.
6. **Outlook.** Each brand's current variance is applied to its December plan, giving an indicative exit ARR vs plan.

## 6. Data model (fictional)

Four brands with monthly figures from January to September 2026 (actuals) and January to December 2026 (plan). Amounts are in EUR thousands.

| Brand | Segment | Scenario built into the data |
|---|---|---|
| Lexgard | Privacy compliance | Stable, tracking plan |
| Optify | Consent management | Consistent outperformance driven by new-logo growth |
| Riskhold | Risk and controls | Widening revenue gap; enterprise new-logo intake below plan |
| Shieldbase | Information security | Late-period churn spike pulling revenue below plan |

The scenarios were designed so the generator has meaningful, mixed signals to explain, which is closer to a real month than a uniformly good or bad one.

**Derived fields:** ARR (12 × MRR), YTD actual vs plan, customer roll-forward (opening + new − churned), group churn weighted by opening customer base, miss and beat streaks, and the three-month variance trend.

## 7. From prototype to production

| Layer | Prototype | Production approach |
|---|---|---|
| Data | Inline JavaScript arrays | SQL views over a warehouse, consolidating each legacy brand's billing and CRM data under harmonised KPI definitions |
| Metrics and rules | Client-side JavaScript | Versioned metric layer (e.g. dbt or SQL), so definitions are governed and reusable |
| Checks | Four reconciliation rules | Automated data-quality tests that block a draft when they fail |
| Narrative | Deterministic template rules | A language model prompted with the computed fact set only, with figure-level validation that every number in the output matches a source value |
| Distribution | Approve button | Approval logged with reviewer and timestamp, then delivery by email or Slack |

A deterministic generator was chosen deliberately for the prototype. It shows the logic transparently, it is fully reproducible, and it defines the fact contract a language model would later be constrained to.

## 8. Limitations

1. The data is synthetic and simplified. It contains no currency effects, revenue-recognition nuance, expansion or contraction revenue, or net revenue retention.
2. Template-based text is less varied than model-generated prose. That is an accepted trade-off for traceability in this version.
3. The run-rate outlook assumes the current variance persists. It is not a forecast model.
4. Causal explanations are intentionally not asserted. They require qualitative input from brand owners.

## 9. Technical notes

1. Single self-contained HTML file. All CSS and JavaScript are inline.
2. Charts are hand-built SVG with no charting library dependency.
3. The only external resource is Google Fonts (Source Serif 4, IBM Plex Sans). System fonts are used as a fallback if it is unavailable.
4. Responsive down to mobile, with light and dark themes, keyboard focus states, and reduced-motion support.
5. Developed iteratively with an AI coding assistant (Claude). The metric definitions, business rules, narrative logic and design direction were specified and reviewed by the author.

## 10. Author

Jan, MSc Business Administration & Digital Business (Copenhagen Business School) and MSc Engineering Management (Poznań University of Technology). [LinkedIn](https://linkedin.com/in/janzagdanski)
