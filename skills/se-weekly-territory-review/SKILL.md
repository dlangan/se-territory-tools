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

#### 2C. Customer 360 Methodology Phase Mapping

**Which deals get a methodology check:**
1. **All opps closing this month + next month** (regardless of amount) — urgency demands it
2. **All opps ≥$100K closing later** — size/importance warrants it
3. Skip opps that are both small (<$100K) AND closing 60+ days out — they'll get picked up as they approach

**Account-level vs. Opp-level assessment:**
- The **5Cs Discovery** is assessed at the **account level** — you discover the customer once, and that feeds all opps on that account
- The **BUILD TRUST and PARTNER checklists** are assessed at the **opp level** — each opp needs its own demo, complexity assessment, commercial terms
- For accounts with multiple opps, show the 5Cs once and then the phase checklist per-opp where they differ

Map each opportunity to its current phase in the Salesforce Customer 360 Methodology (Sales Path for Solution & Industry):

| Phase | Org62 Stages | What It Means | Advance When |
|---|---|---|---|
| **LISTEN** | 01-02 | Gaining deep understanding of customer. Building POV. Getting commitment to engage. | Customer commits to deeper engagement + Discovery Event scheduled |
| **BUILD TRUST** | 03-04-05 | Building credibility through demos, workshops, discovery. Demonstrating differentiation. | Customer validates Connected Vision + identifies Salesforce as vendor of choice |
| **PARTNER** | 06-07-08 | Aligning on Joint Solution. Commercial negotiation. Closing. | Deal signed |
| **SUCCEED** | Post-close | Ensuring customer receives promised value | Ongoing |

**For each opp, assess required activities completion for its current phase:**

**LISTEN phase (Stage 01-02) required activities:**
- [ ] Joint Account Plan with Executive Power Map
- [ ] Solution Use-Case POV (with C360 Connected Vision)
- [ ] Collaborate on Connected Vision with customer
- [ ] Customer Commitment to Deeper Engagement
- [ ] Position Premier/Signature Success Plans
- [ ] 5Cs Discovery (assess each individually):

**5Cs Discovery Checklist:**
| C | Status | Evidence |
|---|---|---|
| **C-level Priorities** | :white_check_mark: / :large_yellow_circle: / :x: | What do the execs care about most? Named priorities from customer's own words. |
| **Challenges** | :white_check_mark: / :large_yellow_circle: / :x: | What obstacles are blocking their goals? Operational pain, process gaps, skill gaps. |
| **Competitive Threats** | :white_check_mark: / :large_yellow_circle: / :x: | Who/what are they competing against in their market? Market pressure driving urgency. |
| **Compelling Events** | :white_check_mark: / :large_yellow_circle: / :x: | Time-bound triggers: contract expirations, regulatory deadlines, budget cycles, go-lives, EOL dates. |
| **Potential Crises** | :white_check_mark: / :large_yellow_circle: / :x: | Risks that escalate if unaddressed: attrition, compliance failures, system outages, revenue leakage. |

For each deal in LISTEN phase, assess how many of the 5Cs have been identified:
- **5/5 identified** → ready to build a strong POV and advance
- **3-4/5 identified** → POV possible but has blind spots — flag what's missing
- **1-2/5 identified** → discovery is incomplete — do not advance to BUILD TRUST
- **0/5 identified** → no real discovery has happened — this is a ghost opp regardless of stage

The 5Cs map directly to SE Scorecard metrics:
- C-level Priorities → Exec Access & Sponsorship (metric 9)
- Challenges → Process & Functional Discovery (metric 1)
- Competitive Threats → Competitive Position (metric 7)
- Compelling Events → Compelling Event (metric 2)
- Potential Crises → Unique Business Value (metric 5) — the "what happens if you do nothing" story

**BUILD TRUST phase (Stage 03-04-05) required activities:**

**Build Trust Checklist:**
| Activity | Status | Evidence Source | What Good Looks Like |
|---|---|---|---|
| **Discovery Workshop conducted** | :white_check_mark: / :large_yellow_circle: / :x: | Slack call notes, calendar events, Google Doc agendas in #ZC channel | Co-creation session with customer stakeholders; documented outcomes and follow-ups |
| **Solution Demo / Holodeck delivered** | :white_check_mark: / :large_yellow_circle: / :x: | Slack posts ("demo went well"), CRM activity, demo prep threads | Tailored demo mapped to discovery findings, not a generic product tour |
| **Connected Vision evolved & validated** | :white_check_mark: / :large_yellow_circle: / :x: | Google Doc/Slides shared in Slack, customer feedback documented | Customer has seen, commented on, and agreed to the vision — not just internal |
| **Complexity Assessment done** | :white_check_mark: / :large_yellow_circle: / :x: | Integration landscape discussed, user counts confirmed, timeline mapped | Data sources, integrations, user volumes, and go-live timeline documented |
| **Business Value quantified (BVS engaged)** | :white_check_mark: / :large_yellow_circle: / :x: | BVS DSR filed, ROI doc shared, financial model referenced | Customer-specific ROI with quantified outcomes (not generic "you'll save time") |
| **Competitive differentiation delivered** | :white_check_mark: / :large_yellow_circle: / :x: | Competitive positioning in Slack, CI team engaged, battle card used | Customer understands why Salesforce vs. specific named alternatives |
| **Vendor-of-choice signal received** | :white_check_mark: / :large_yellow_circle: / :x: | Customer language in call notes: "we want Salesforce," verbal commit, shortlist of 1 | Explicit customer statement — not seller assumption |

For each deal in BUILD TRUST phase, assess completion:
- **6-7/7 complete** → ready to advance to PARTNER, commercial conversation appropriate
- **4-5/7 complete** → progressing well but gaps remain — address before advancing
- **2-3/7 complete** → deal is premature at this stage — either regress or accelerate missing activities
- **0-1/7 complete** → stage is wrong — this deal hasn't earned BUILD TRUST regardless of what CRM says

**PARTNER phase (Stage 06-07-08) required activities:**

**Partner Checklist:**
| Activity | Status | Evidence Source | What Good Looks Like |
|---|---|---|---|
| **Mutual Close Plan / Joint Eval Plan** | :white_check_mark: / :large_yellow_circle: / :x: | Doc shared in Slack, milestones with dates, customer co-owns timeline | Named milestones, owners on both sides, dates agreed — not a seller-only internal doc |
| **Commercial terms in motion** | :white_check_mark: / :large_yellow_circle: / :x: | Quote exists in Org62, pricing call logged, DocuSign sent | Customer has seen pricing and engaged on it — not just a quote sitting in a drawer |
| **Implementation partner confirmed** | :white_check_mark: / :large_yellow_circle: / :x: | Partner named in Slack, partner on calls, SOW discussed | Customer has met the partner, scope is discussed, partner is committed |
| **Legal / Procurement engaged** | :white_check_mark: / :large_yellow_circle: / :x: | Procurement contact identified, MSA/security review started, legal questions asked | Procurement is aware and active — not "we'll send it to procurement when ready" |
| **Success Plan confirmed** | :white_check_mark: / :large_yellow_circle: / :x: | Premier/Signature tier selected, included in quote, customer understands value | Customer chose a tier based on their complexity/risk, not just seller bundling it in |
| **Executive alignment (both sides)** | :white_check_mark: / :large_yellow_circle: / :x: | Salesforce exec sponsor assigned, customer exec in room for commercial discussion | Both sides have executive-level ownership of the deal's success — not just AE-to-champion |

For each deal in PARTNER phase, assess completion:
- **5-6/6 complete** → deal is real, close is mechanical — focus on timeline and removing blockers
- **3-4/6 complete** → deal is progressing but has commercial/procurement gaps that could stall at the last mile
- **1-2/6 complete** → deal is premature at PARTNER — significant risk of "stuck at Stage 06" syndrome
- **0/6 complete** → stage is fiction — this deal has been advanced without earning it

**"Stuck at Stage 06" warning:** Deals that reach PARTNER without completing BUILD TRUST activities (no demo, no business value, no competitive win) are the most common source of late-stage slips. They look like they're about to close but have no foundation. Flag these aggressively.

**In the canvas, flag methodology gaps as:**
- :white_check_mark: Activity completed (evidence exists)
- :large_yellow_circle: Activity in progress (partial evidence)
- :x: Activity not started (no evidence)

This assessment feeds into the Structural Insights section — patterns like "80% of Stage 02 deals have no POV" or "no deal in the portfolio has a completed Connected Vision" are high-value coaching signals.

#### 2D. Structural Pattern Detection

Look across the full portfolio for:
- Concentration risk (single account dominance)
- Sequencing violations (downstream deals advanced before parent)
- Activity gaps (CRM vs Slack mismatch)
- Post-demo follow-up failures (demos delivered, then silence)
- SE attachment gaps (no scorecard at Stage 02+)
- Close date discipline (Stage 02 with near-term close dates)
- Ghost pipeline (opps with zero activity ever)
- **Methodology gaps** — deals at advanced stages without completing the required activities for their phase (e.g., Stage 04 with no demo delivered, Stage 02 with no POV, Stage 06 with no Mutual Close Plan)
- **Premature stage advancement** — deals that have been moved forward in stage without earning it through methodology activities (the most dangerous pipeline fiction)

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

# :compass: Methodology Phase Check

| Opportunity | Phase | Stage | Key Gap | Required Activity Missing |
| --- | --- | --- | --- | --- |
| [Opp Name] | LISTEN | 02 | No POV delivered | Solution Use-Case POV not created |
| [Opp Name] | BUILD TRUST | 04 | No demo delivered | Solution Demo / Holodeck not scheduled |

For each ≥$100K opp (or any opp where methodology gaps are blocking advancement), show:
- Current phase based on stage
- The most critical missing required activity for that phase
- Whether the deal SHOULD be in its current phase (e.g., Stage 04 with no discovery = premature advancement)
- **Recommended action to strengthen and progress the deal**

Flag deals where stage is ahead of methodology completion — these are the most dangerous because they look healthy in the forecast but lack the foundation to close.

### Methodology Strengthening Recommendations

For each gap identified, recommend a specific action mapped to the missing activity:

**LISTEN phase gaps → Recommended actions:**
| Missing Activity | Recommended Action |
| --- | --- |
| No Joint Account Plan | Schedule 30-min internal account planning session with extended team (SE, CSG, Partner) |
| No POV | Draft Solution Use-Case POV using account research + 5Cs; share with customer sponsor for validation |
| No Connected Vision | Build a 1-slide Connected Vision showing how Salesforce products connect to their stated priorities |
| No Customer Commitment | Ask for a specific next step: "Can we schedule a 60-min discovery workshop with your team?" |
| 5Cs not identified | Run a 5Cs research sweep (Slack, web, 10-K, earnings calls) and draft hypothesis for AE to validate |
| No Success Plan positioned | Introduce Premier/Signature in context of implementation risk reduction — not as an upsell |

**BUILD TRUST phase gaps → Recommended actions:**
| Missing Activity | Recommended Action |
| --- | --- |
| No Discovery Workshop | Propose a co-creation workshop agenda tied to their top 2-3 stated priorities |
| No Demo / Holodeck | Schedule a tailored demo mapped to discovery findings — not a generic product tour |
| Connected Vision not evolved | Update the Connected Vision with customer-validated language from discovery, present back for confirmation |
| No Complexity Assessment | Review integration landscape, data volume, user count, and timeline — flag risks early |
| No vendor-of-choice signal | Ask directly: "What would need to be true for you to choose Salesforce?" — surface decision criteria |
| Success Plan not reinforced | Tie Success Plan to implementation complexity: "Given X integrations and Y users, Premier/Signature de-risks your go-live" |

**PARTNER phase gaps → Recommended actions:**
| Missing Activity | Recommended Action |
| --- | --- |
| No Mutual Close Plan | Draft a joint evaluation plan with milestones, owners, and dates — present to customer for co-ownership |
| No Implementation Partner | Introduce 2-3 SI options based on account size, industry, and complexity — facilitate partner intro call |
| Commercial terms not started | AE to initiate pricing conversation anchored to business value quantified in BUILD TRUST phase |
| Success Plan not confirmed | Finalize Success tier selection with customer as part of commercial negotiation |

These recommendations should be included in the canvas under each deal's methodology row and also feed into the "This Week's Non-Negotiables" prioritization.

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
