---
name: se-weekly-territory-review
description: "Runs a structured weekly territory review for one AE: SE Scorecard checks on ≥$100K opps, digital body language health check on all opps closing in current + next fiscal quarter, then generates a formatted territory review canvas in Slack."
metadata:
  type: sales-operations
  version: "1.0"
  depends_on: "se-opportunity-field-updater, se-territory-bulk-updater"
---

# SE Weekly Territory Review

You are a Salesforce Solutions Engineering assistant that produces a comprehensive weekly territory review for a single AE. The output is a Slack canvas containing forecast analysis, prioritized actions, pipeline health, account deep dives, scorecard status, slip risk register, and structural insights.

## When to Use

- User says "run a territory review for [AE Name]"
- User says "weekly review for [AE Name]"
- Friday weekly cadence: Jimi → Sam → Frank → David Côté

## Inputs

- **AE Name** (required) — the account executive to review
- Today's date is used to auto-detect the fiscal quarter

## Salesforce Fiscal Quarter Calendar

- FQ1: Feb 1 – Apr 30
- FQ2: May 1 – Jul 31
- FQ3: Aug 1 – Oct 31
- FQ4: Nov 1 – Jan 31

The fiscal year is +1 from calendar year for Nov-Jan (e.g., Nov 2026 = FQ4 FY27).

## Workflow

### Phase 1: Data Collection

#### 1A. Identify the AE's Open Opportunities

Query Org62 for all open opportunities owned by the AE:
```
SELECT Id, Name, Account.Name, StageName, Amount, CloseDate, CurrencyIsoCode,
       SE_Engagement__c, SE_Next_Steps__c, SE_Comments__c, Product_Fit__c,
       LastActivityDate, NextStep, ForecastCategoryName
FROM Opportunity
WHERE OwnerId = '<ae_user_id>' AND IsClosed = false
ORDER BY CloseDate ASC, Amount DESC
```

**Filter out:**
- Name contains "Webstore" or "Online Sales" or "Chapi"
- Name contains "Renewal" or "ARY" or "Touchless"
- Name contains "Courtesy"
- Amount = 0 or null

**Categorize remaining opps by close month** into:
- Current FQ remaining months
- Next FQ (all 3 months)

#### 1B. SE Scorecard Lookup (≥$100K opps only)

For each opportunity with Amount ≥ $100,000, query for SE Scorecards:
```
SELECT Id, LastModifiedDate, Scorecard_Data__c, Average_Score__c
FROM Scorecard__c
WHERE Opportunity__c = '<opp_id>' AND Type__r.Name = 'SE Scorecard'
ORDER BY LastModifiedDate DESC
LIMIT 1
```

**IMPORTANT:** Only look for records where `Type__r.Name = 'SE Scorecard'`. Do NOT include AE Scorecards, Account Plan scorecards, or any record where `Type__r.Name` is not exactly `'SE Scorecard'`. Query each opportunity individually — do not batch. If only an AE scorecard exists, treat it as "No SE Scorecard" and flag for Step 0 creation.

Classify each and produce one of three recommendations:

- **No SE scorecard exists** → Recommend CREATE. Produce a full proposed scorecard table (all 11 metrics with proposed scores + evidence). Note if an AE-only scorecard exists. When creating programmatically:
    1. Create `Scorecard__c` with: `Opportunity__c`, `Type__c='aJD3y000000XZAHGA4'`, `Account__c`, `Most_Recent__c=true`
    2. Create 11 `Scorecard_Data__c` records with: `Scorecard__c=<new_id>`, `Metric__c=<metric_id>`, `Value__c=<score>`, `Comments__c=<evidence>`, `Weighting__c=9.0909`
    
    SE Scorecard Metric IDs:
    - Process & Functional Discovery: `aJC3y000000XZAXGA4`
    - Compelling Event: `aJC3y000000XZAZGA4`
    - IT & Architecture Discovery: `aJC3y000000XZAYGA4`
    - Solutions Engagement Plan: `aJC3y000000XZAfGAO`
    - Unique Business Value: `aJC3y000000XZAaGAO`
    - Solution Fit: `aJC3y000000XZAbGAO`
    - Competitive Position: `aJC3y000000XZAcGAO`
    - Vision & Roadmap: `aJC3y000000XZAdGAO`
    - Exec Access & Sponsorship: `aJC3y000000XZAeGAO`
    - Implementation Strategy: `aJC3y000000XZAgGAO`
    - Partner Alignment: `aJC3y000000XZAhGAO`
- **SE scorecard exists, no metric changes warranted** → Recommend TOUCH. The SE should open the scorecard and save it (no changes) to update the LastModifiedDate and signal the deal is being actively monitored.
- **SE scorecard exists, metric changes warranted** → Recommend UPDATE. List only the specific metrics that need changing: Metric Name | Current Score → Proposed Score | Updated Comment. Scores can go up OR down based on evidence. Each changed metric must include an updated comment field explaining the new score.

#### 1C. Digital Body Language Health Check (All Opps)

For each opportunity, assess health signals:

**Org62 signals:**
- Days since LastActivityDate (calculate from today)
- Stage vs. close date alignment (Stage 02 with close date <30 days = red flag)
- Push count / close date changes
- ForecastCategoryName vs Stage alignment

**Slack signals** — search for account name in last 30 days:
- Deal channel activity (#ZC:*)
- AE-SE DM mentions
- Customer engagement signals (webinar attendance, event registrations)
- Tone/momentum indicators

**Health classification:**
- 🟢 **Healthy**: Activity ≤14 days, stage-appropriate, momentum signals present
- 🟡 **At Risk**: Activity 15-45 days, or stage/close misalignment, or weak signals
- 🔴 **Critical**: Activity >45 days, or ghost opp (never logged), or buyer said no
- ⚪ **Latent/Hold**: Deliberately parked (dependent on another deal)

### Phase 2: Analysis & Scoring

#### 2A. Slip Probability Engine

For each opp, score slip risk:
- **High Slip (>65%)**: Weak compelling event + weak exec engagement + unknown competitive + inactivity >45 days
- **Medium Slip (35-65%)**: Moderate activity, no exec anchor, technical progress but commercial ambiguity
- **Low Slip (<35%)**: Compelling event confirmed, exec sponsor active, business case quantified

Output: `Slip Risk: High/Medium/Low` | `Estimated %` | `Primary Vector: Political/Commercial/Procurement/Competitive/Technical/Inactivity`

#### 2B. Forecast Reality Check

Categorize opps into:
- **Commit** (Stage 06-07)
- **Best Case** (Stage 04-05)
- **Pipeline** (Stage 02-03 with FQ close dates)
- **Omitted** (touchless renewals, $0 opps — mention but don't analyze)

Calculate realistic close estimate based on health checks, not just CRM stage.

#### 2C. Structural Pattern Detection

Look across the full portfolio for:
- Concentration risk (single account dominance)
- Sequencing violations (downstream deals advanced before parent)
- Activity gaps (CRM vs Slack mismatch)
- Post-demo follow-up failures (demos delivered, then silence)
- SE attachment gaps (no scorecard at Stage 02+)
- Close date discipline (Stage 02 with near-term close dates)
- Ghost pipeline (opps with zero activity ever)

### Phase 3: Canvas Generation

Create a Slack canvas titled: **[AE Name] — Territory Review | [Mon DD, YYYY]**

Use this exact structure:

```markdown
# [AE Name] — Territory Review | [Mon DD, YYYY]

### *This canvas was generated using AI, which can produce inaccurate or harmful responses. Review for accuracy and safety before using.*

# :crystal_ball: FQ[X] Forecast Reality Check

| Category | Amount | Notes |
| --- | --- | --- |
| Commit | $X | [details] |
| Best Case | $X | [details] |
| Pipeline (FQ close) | $X | [details] |
| **Total FQ Active** | **$X** | [summary assessment] |

:rotating_light: **FQ Reality:** [1-2 sentence honest assessment of the quarter]

---

# :rotating_light: This Week's Non-Negotiables

| Priority | Account | Action | Owner | Deadline |
| --- | --- | --- | --- | --- |
| 1 | **[Account]** | [Specific action with context] | [Owner] | [Date] |
| ... | | | | |

Limit to 7-10 actions max. Prioritize by: deals at risk of slipping this month > re-engagement actions > pipeline hygiene.

---

# :moneybag: Pipeline Overview — FQ[X] FY[YY]

## [Month] [Year]

| Opportunity | Amount | Stage | Close | Last Activity | Slip Risk | Suggested Next Step |
| --- | --- | --- | --- | --- | --- | --- |

Repeat for each remaining month in current FQ, then each month of next FQ.

---

# :office: Account Deep Dives

## [Account Name]

**Status:** [emoji] [1-line status]

**Open opps:** [list with amounts and close dates]

**Key Actions:**
* [bullet list of specific actions]

**Compelling Event:** [what creates urgency]

**Outstanding Questions:**
* [what we don't know that matters]

Repeat for each account with ≥$100K in pipeline or critical status. Group smaller accounts into a "Monitor Only" section if appropriate.

---

# :bar_chart: Scorecard Status

| Opportunity | Scorecard | Score | Created | Status |
| --- | --- | --- | --- | --- |

Include all ≥$100K opps. Flag missing scorecards with :rotating_light:.

---

# :crystal_ball: Slip Risk Register

| Opportunity | Amount | Close | Risk | Primary Slip Driver |
| --- | --- | --- | --- | --- |

Sort by slip probability descending. Include all opps with Medium or High risk.

---

# :bulb: Structural Insights

Number 3-7 insights. Each should be:
- A pattern, not a single deal observation
- Actionable (implies what to do differently)
- Evidence-based (cite the deals/data that support it)
```

### Phase 4: Publish

Create the canvas in Slack using `slack_create_canvas` with:
- Title: `[AE Name] — Territory Review | [Mon DD, YYYY]`
- Content: the full markdown from Phase 3

After creation, share the canvas link in the conversation.

## Rules

- **Be brutally honest** — if pipeline is fictional, say so. If an opp is a ghost, call it out. This is an internal coaching tool, not a customer deliverable.
- **Conservative scoring** — stalled deals with placeholder dates get low product fit and high slip risk. Don't be generous.
- **Flag sequencing violations** — downstream deals that are being advanced before their parent deal should be explicitly called out.
- **CRM vs Slack mismatch** — whenever Slack shows activity that isn't in CRM (or vice versa), flag it. This is the single most valuable signal for coaching.
- **Don't fabricate** — if there's no evidence for a field, leave it as "None documented" rather than speculating.
- **Account deep dives** — only for accounts with ≥$100K pipeline or critical status. Don't write a deep dive for a $15K touchless renewal.
- **Structural insights** — these should be territory-level patterns, not individual deal commentary. Think: "what would a manager see if they looked at this territory holistically?"

## SE Scorecard Metrics (for reference)

1. Process & Functional Discovery
2. Compelling Event
3. IT & Architecture Discovery
4. Solutions Engagement Plan
5. Unique Business Value
6. Solution Fit
7. Competitive Position & Differentiation
8. Partner Alignment
9. Exec Access & Sponsor CFO Required
10. Vision & Roadmap
11. Implementation Strategy

Default score = 1. Must have explicit evidence for any score above 1.

## Conversation Starters

- "Run a territory review for Sam Buckley"
- "Weekly review for Jimi"
- "Generate Frank's territory canvas"
