---
name: se-event-recommender
description: "Deep-dive account-level activity recommender. Given a specific account, scans ALL future events + on-demand webinars, matches to open opportunities, entitlements, competitive context, and methodology needs. Produces a comprehensive menu of content options to advance each deal. Canvas output: Account → Opportunity → Full Menu of Matching Activities."
metadata:
  type: sales-operations
  version: "2.0"
  depends_on: "se-entitlement-decoder, se-competitive-intel-updater, se-deal-coach"
---

# SE Event & Activity Recommender

You produce a comprehensive menu of ALL available content and events that can advance a specific account's deals. This isn't a "top 3 picks" — it's the full buffet: every future event, every on-demand webinar, every Salesforce+ recording that matches this account's context, organized by opportunity and scored by relevance.

The goal: when an SE or AE sits down to plan how to advance an account, they have a complete menu of options — not just what's happening this week, but everything available to them.

## When to Use

- "What activities match [Account]?"
- "Give me the full content menu for Ecobee"
- "What can I send/register [Account] for?"
- "Event and content recommendations for [Account]"
- Before any strategic account conversation — know what content artillery is available
- Alongside Deal Coach — "here's what to do next" + "here's the content that supports it"

## Inputs

- **Account** (required) — specific account name, ID, or org62 URL

## Scope

- **Future events:** From today through the end of the current Salesforce fiscal year (Jan 31)
  - Fiscal year calendar: FQ1 Feb-Apr, FQ2 May-Jul, FQ3 Aug-Oct, FQ4 Nov-Jan
  - If today is Jun 3, 2026 → scan through Jan 31, 2027 (end of FY27)
- **On-demand content:** Last 6 months of evergreen webinars/recordings still available
- **Note gaps:** When scanning far out, campaigns may not exist yet. Flag: "Beyond [date], few campaigns published. Re-run in 4-6 weeks to capture newly created events."

## Workflow

### Phase 1: Build Activity Inventory (Supply Side)

Scan ALL campaigns from today through end of fiscal year. This is the "menu" of available content.

**Future live events:**
```
SELECT Id, Name, Description, StartDate, Type
FROM Campaign
WHERE StartDate >= TODAY AND StartDate <= <fiscal_year_end>
ORDER BY StartDate ASC
```

**On-demand content (last 6 months):**
```
SELECT Id, Name, Description, StartDate, Type
FROM Campaign
WHERE (Name LIKE '%On-Demand%' OR Name LIKE '%OnDemand%' OR Description LIKE '%on-demand%'
       OR Description LIKE '%available now%' OR Description LIKE '%watch anytime%')
  AND StartDate >= LAST_N_MONTHS:6
ORDER BY StartDate DESC
```

**For each campaign, parse and classify:**
- Parent Event (World Tour, Dreamforce, Connections, TDX, TC, BYOE, SIC, Standalone)
- Session Type (Workshop, Breakout, Executive, SIC, Demo, Webinar, Main Reg)
- Location (city + country flag)
- Product Category (Agentforce, Service, FSL, Data Cloud, Tableau, Marketing, MuleSoft, Slack, etc.)
- Engagement Level (Highest/High/Medium/Low → exclude Low)

**Filter out noise:** Holding campaigns, email attribution, paid/organic social, onsite leads, display, SEM/SEO.

**Flag inventory gaps:** Note where campaign creation thins out (typically 2-3 months ahead). Add: "Beyond [date], campaign inventory is sparse. Re-run in [X] weeks to capture new events as they're published."

---

### Phase 2: Build Account Context (Demand Side)

Gather everything about THIS account to identify what content matches:

#### 2A. Open Opportunities
```
SELECT Id, Name, Amount, StageName, CloseDate, Product_Fit__c,
       SE_Engagement__c, SE_Comments__c, SE_Next_Steps__c
FROM Opportunity
WHERE AccountId = '<account_id>' AND IsClosed = false AND Amount > 0
ORDER BY Amount DESC
```

For each opportunity, note:
- Product family (what category of session would advance this deal?)
- Methodology phase (LISTEN → discovery workshops; BUILD TRUST → demos/deep dives; PARTNER → exec events)
- SE Comments (what specific topics are in play?)

#### 2B. Current Contract SKU → Entitlement Map (via se-entitlement-decoder)

Run the full entitlement decoder:
- Pull active contracts with line items
- Resolve each SKU through ProductLicenseMap → LicenseDefinition
- Identify what they ALREADY have access to but may not be using

**Why this matters for recommendations:**
- Products they OWN but aren't using → recommend adoption/deepening workshops ("You have OmniStudio entitled — here's a workshop to activate it")
- AI entitlements already included → recommend AI sessions as "learn to use what you own" not "buy something new"
- Products they own AND use well → recommend advanced/next-level content

#### 2C. Case Insight Analysis (via se-case-insight-analyzer)

Run case insights to identify:
- Technical themes in support cases → recommend sessions that address recurring issues
- Adoption gaps signaled by case patterns → recommend enablement content
- Integration challenges → recommend MuleSoft/platform sessions

#### 2D. Competitive Intelligence

Pull from CI record:
- What competitors are in play? → recommend sessions that differentiate against those competitors
- What categories is SF incumbent in? → recommend deepening/advanced sessions
- What's being displaced? → recommend sessions that validate the displacement decision

#### 2E. Slack Intelligence — Interests NOT Yet Captured as Opportunities

Search Slack (deal channel + DMs + broad) for:
- Topics the customer has expressed interest in that DON'T have a matching open opp
- Questions they've asked that indicate latent demand
- Webinar attendance or event participation that signals emerging interest
- Technology mentions that suggest future needs

**This surfaces "pre-pipeline" interests** — things the customer cares about that haven't been formalized into an opportunity yet. These become "Additional Recommendations" that could CREATE new pipeline, not just advance existing.

Examples:
- Customer asked about Slack integrations in a call → no Slack opp exists → recommend Slack session
- Contact attended 3 Data Cloud webinars → no Data Cloud opp exists → recommend Data Cloud workshop + flag as potential new opp

#### 2F. SE Field Context

From the opportunity SE fields:
- SE_Engagement__c → what phase are we in?
- SE_Comments__c → what's the current context?
- Product_Fit__c → what's the solution alignment?

### Phase 3: Match Inventory to Context

**Two queries — future events + on-demand content:**

**Future-dated events (live sessions, workshops, in-person):**
```
SELECT Id, Name, Description, StartDate, Type
FROM Campaign
WHERE StartDate >= TODAY AND StartDate <= <end_date>
ORDER BY StartDate ASC
```

**On-demand webinars (past-dated but still available):**
```
SELECT Id, Name, Description, StartDate, Type
FROM Campaign
WHERE (Name LIKE '%On-Demand%' OR Name LIKE '%OnDemand%' OR Name LIKE '%on demand%'
       OR Description LIKE '%on-demand%' OR Description LIKE '%available now%'
       OR Description LIKE '%watch anytime%')
  AND StartDate >= LAST_N_MONTHS:6
ORDER BY StartDate DESC
```

On-demand content is **evergreen** — it doesn't expire. A webinar recorded 3 months ago is still a valid recommendation if it matches the customer's current context. Flag these differently in the canvas:
- Future live events: show date + location + registration action
- On-demand content: show as "Available Now" with a share action (send link to customer) rather than a register action

**Parse campaign naming convention to identify Event + Session Type:**

Campaign names follow a structured pattern:
`[Region]-[Country]-[Channel]--[Event/Description]--[FiscalPeriod]`

Or for PGM events:
`[Region]-[Country]-PGM-Events:[Cloud/Track]:[Session Title]:[City]:[Date]:[Tactic]:FY[XX]`

#### Identify the Parent Event

| Pattern in Campaign Name | Event |
|---|---|
| `World Tour` or `WT` or `AFWT` | Agentforce World Tour |
| `Dreamforce` or `DF` | Dreamforce |
| `Connections` or `CNX` | Connections |
| `TDX` or `TrailblazerDX` | TDX (TrailblazerDX) |
| `TC26` or `Tableau Conference` | Tableau Conference |
| `BYOE` (without WT/DF context) | Bring Your Own Event (standalone workshop) |
| `SIC-[number]` | Salesforce Innovation Center (1:1 strategic meeting) |
| No event pattern | Standalone campaign (webinar, content, digital) |

#### Identify the Location (In-Person Events)

The campaign naming convention encodes city/location:
- PGM pattern: `...[Cloud]:[Session Title]:[City]:[Date]...` — city is explicit
- BYOE pattern: `BYOE [City] [Topic]` — city follows BYOE
- Web Form pattern: `[Date] [City] [Event Name]` — city after date
- SIC pattern: region prefix `AMER-CA` / `AMER-US` / `EMEA-UK` etc. indicates country

| Pattern | Location |
|---|---|
| `:Toronto:` or `TOR` or `Toronto` in name | Toronto, ON |
| `:San Francisco:` or `SF` | San Francisco, CA |
| `:New York:` or `NYC` | New York, NY |
| `:London:` or `LON` | London, UK |
| `:Sydney:` or `SYD` | Sydney, AU |
| `:Chicago:` | Chicago, IL |
| `:Vancouver:` | Vancouver, BC |
| `:Montreal:` or `MTL` | Montreal, QC |
| Region prefix `AMER-CA` | Canada (city in name) |
| Region prefix `AMER-US` | United States (city in name) |
| Region prefix `EMEA-` | Europe/Middle East/Africa |
| Region prefix `AP-` | Asia Pacific |
| `WW` prefix | Worldwide / Virtual |

**Location informs but does NOT filter recommendations:**
- **Always surface relevant sessions regardless of location** — the AE decides if the customer will travel
- Flag in-person sessions with a location indicator so the AE can assess feasibility:
  - :maple_leaf: Canadian city (local — easy for our accounts)
  - :us: US city (may require travel)
  - :earth_americas: International (significant travel)
  - :globe_with_meridians: Virtual / On-Demand (no travel needed)
- For flagship events (Dreamforce, Connections, TDX) — ALWAYS surface relevant sessions. These are events customers plan travel for months in advance. The recommendation itself might be what prompts the registration.

#### Identify the Session Type

| Pattern | Session Type | Engagement Level |
|---|---|---|
| `Main Reg` | Main Event Registration | Awareness — registered for the event |
| `Breakout` or `Session` | Breakout Session | Active — chose a specific topic |
| `BYOE` or `Workshop` | Hands-On Workshop | High Intent — investing time to learn |
| `Keynote` | Keynote | Passive — general attendance |
| `CxO` or `Executive` or `Dinner` | Executive Experience | Exec Engagement — relationship building |
| `SIC-[number]` | SIC Consultation | Highest Intent — 1:1 strategic meeting |
| `Kiosk` or `Booth` or `Demo Station` | Booth/Demo Interaction | Curiosity — stopped to look |
| `Onsite Leads` or `Scan` or `Hubbl` | Badge Scan / Lead Capture | Passive — walked by / got scanned |
| `After Party` or `Social` | Social/Networking | Relationship — not product-specific |
| `Ancillary Registration` | Ancillary Event (add-on) | Active — registered for extras beyond main event |
| `Consultation` | Agentforce Consultation | High Intent — requested 1:1 advice |
| `Web Form` (without event context) | Digital/Webinar | Variable — could be registration or attendance |
| `Email` or `Paid Social` or `Organic Social` | Marketing channel tracking | EXCLUDE — not a session, just attribution |
| `HOLDING` or `do not call` (in description) | Holding/Tagging campaign | EXCLUDE — not a customer activity |

#### Session Type Priority for Recommendations

Only recommend sessions with engagement level **Active or higher**:

| Priority | Session Types | Why |
|---|---|---|
| :star: Highest | SIC Consultation, CxO Dinner, BYOE Workshop | Direct strategic engagement — these move deals |
| :red_circle: High | Breakout Session, Agentforce Consultation, Demo Station | Active choice — customer invested time to engage with a topic |
| :large_yellow_circle: Medium | Main Registration, Ancillary Registration, Kiosk | Awareness signal — registered but hasn't attended yet |
| :white_circle: Low (exclude from recommendations) | Email, Paid/Organic Social, Onsite Leads, Holding | Marketing attribution only — not meaningful for SE targeting |

**Filter logic:** INCLUDE campaigns where Session Type is Highest/High/Medium. EXCLUDE Low.

**Categorize each session by product/topic:**

| Pattern in Campaign Name/Description | Product Category |
|---|---|
| Agentforce, Agent, AI, Einstein, GenAI | Agentforce / AI |
| Service, Contact Center, Voice, Digital Engagement | Service Cloud |
| Sales Cloud, Pipeline, Forecasting, Revenue | Sales Cloud |
| Field Service, FSL, Dispatcher, Technician, Mobile Worker | Field Service |
| Data Cloud, Data 360, CDP, Segmentation, Golden Record | Data Cloud |
| Tableau, Analytics, BI, Dashboard, Reporting | Tableau / Analytics |
| Marketing, MCAE, Journey, Email, Engagement | Marketing Cloud |
| MuleSoft, Integration, API, iPaaS | MuleSoft |
| Slack, Collaboration, Workflow, Slackbot | Slack |
| Commerce, B2B, B2C, Storefront | Commerce Cloud |
| Manufacturing, Automotive, Energy, Utilities | Industry - Manufacturing |
| Healthcare, HLS, Life Sciences, Pharma | Industry - Healthcare |
| Financial, Banking, Insurance, Wealth | Industry - FSI |
| Shield, Security, Encryption, Compliance, Backup | Security / Trust |
| Platform, Flow, OmniStudio, Development, DevOps | Platform / Dev |
| Success, Premier, Signature, Adoption | Customer Success |
| CxO, Executive, Leadership, Strategy | Executive |

### Phase 3 (continued): Score and Match

For each account, score every future session on relevance:

**Scoring criteria:**

| Signal | Score Weight | Logic |
|---|---|---|
| **Direct opp match** | +5 | Session product category matches an open opportunity's product family |
| **Entitlement deepening** | +3 | Session covers a product they own but may be underutilizing |
| **Competitive displacement** | +4 | Session validates displacing a named competitor |
| **Slack signal match** | +3 | Topic recently discussed in deal channel |
| **SE Comments match** | +2 | Session topic mentioned in SE field context |
| **Stage acceleration** | +3 | Session type matches methodology need (e.g., demo session when deal needs BUILD TRUST) |
| **Executive session for exec-level deal** | +4 | CxO event when deal needs exec engagement |
| **Adjacent product expansion** | +2 | Session for a product not in pipeline but logical next step |

Only recommend sessions scoring **5+** (strong match) or **3-4** (moderate — goes in "Additional").

### Phase 4: Generate Canvas

Take the matched inventory and produce the canvas, organized by:
1. **Per Opportunity** — activities that directly advance a specific deal
2. **Entitlement Activation** — sessions that help them use what they already own
3. **Pre-Pipeline / Emerging Interest** — content matching Slack signals that don't have opps yet (could CREATE new pipeline)
4. **On-Demand Library** — evergreen content to share anytime

Create a Slack canvas titled: **[Account Name] activity recommendations dd/Mon/yy**

(e.g., "Ecobee activity recommendations 03/Jun/26")

```markdown
# [Account Name] Activity & Content Menu | dd/Mon/yy

### *This canvas was generated using AI, which can produce inaccurate or harmful responses. Review for accuracy and safety before using.*

**Account:** [Name] | **AE:** [Name] | **SE:** [Name]
**Open Pipeline:** $[X] across [Y] opps | **Generated:** [Date] | **Looking ahead:** [X] days
**What they own:** [Key SKUs/edition — from entitlement decoder]
**Competitors in play:** [From CI record]

---

# :office: [Account Name] | Pipeline: $[X] across [Y] opps

## :dart: [Opportunity Name] — $[Amount] | Stage [X] | Close [Date]

**Context:** [1-line from SE Comments or deal summary]
**Matching sessions:**

| Session | Event | Type | Location | Date | Why This Matches | Score | Action |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Session Name] | [World Tour / DF / BYOE] | [Workshop / Breakout / CxO] | [Toronto / Virtual / SF] | [Date] | [Specific reason tied to this opp — e.g., "Agentforce Service workshop directly maps to $171K AF CC pilot"] | [X/10] | Register [Contact Name] |
| [Session Name] | [Event] | [Type] | [Location] | [Date] | [Reason] | [X/10] | Register [Contact Name] |

## :dart: [Opportunity Name 2] — $[Amount] | Stage [X]

**Context:** [context]
**Matching sessions:**

| Session | Date | Why | Score | Action |
| --- | --- | --- | --- | --- |

## :bulb: Additional Recommendations (Not Tied to Active Opps)

*These sessions match the account's profile or entitlements but don't align to a specific pursuit. Could surface new opportunities.*

| Session | Event | Type | Location | Date | Why | Relevance |
| --- | --- | --- | --- | --- | --- | --- |
| [Session] | [Event] | [Type] | [Location] | [Date] | [e.g., "They own OmniStudio but aren't using it — this workshop could activate it"] | Adoption deepening |
| [Session] | [Event] | [Type] | [Location] | [Date] | [e.g., "Manufacturing industry session — matches their vertical"] | Industry alignment |

## :tv: On-Demand Content to Share

*Evergreen webinars and recordings that match this account's context. No registration needed — send the link.*

| Content | Topic | Why It Matches | Action |
| --- | --- | --- | --- |
| [Webinar/Recording Name] | [Product category] | [Specific reason — e.g., "Customer Zero webinar validates Shield investment. Trojan has Shield opp open."] | Share link with [Contact] |
| [Content Name] | [Topic] | [Reason] | Share link with [Contact] |

---

# :office: [Next Account] | Pipeline: $[X]

[Repeat structure]

---

# :clipboard: Summary: Registration Actions

| Account | Contact | Session | Date | Priority |
| --- | --- | --- | --- | --- |
| [Account] | [Contact Name] | [Session] | [Date] | :red_circle: High / :large_yellow_circle: Medium |

*Priority: High = direct opp match + deal in active stage. Medium = deepening or adjacent.*

---

# :bar_chart: Event Coverage Overview

| Account | Opps | Sessions Matched | Top Product Interest | Contacts to Register |
| --- | --- | --- | --- | --- |
| [Account] | [X] | [Y] | [Product] | [Names] |

---

# :file_folder: Sessions Scanned

**Event:** [Name] | **Total sessions found:** [X]
**Date range:** [Start] to [End]
**Product categories represented:** [list]
**Sessions that matched at least one account:** [Y] of [X]
```

## Who to Register (Contact Selection)

For each recommendation, suggest which **contact** to register based on:

1. **Contacts on the opportunity** (Contact Roles) — prioritize these
2. **Recent campaign engagement** — contacts who've attended similar sessions before
3. **Title/role alignment** — CxO sessions → VP+ contacts; technical sessions → IT/admin contacts
4. **Slack mentions** — contacts discussed recently in deal channels

If no specific contact is identifiable, flag: "Identify attendee — no contact role set on this opp."

## Rules

- **This is a DEEP DIVE — show everything relevant.** Don't limit to top 3. Show the full menu: every future event, every on-demand piece, every Salesforce+ recording that matches. The SE/AE decides what to use — give them the complete arsenal.
- **Organize by opportunity first, then by "Additional."** Every opp gets its own section with all matching content. Content that doesn't match a specific opp but matches the account's profile goes in "Additional Recommendations."
- **Connect to methodology.** If a deal is in LISTEN and needs a POV, recommend a workshop that could serve as the "deeper engagement" the methodology requires. If BUILD TRUST needs a demo, recommend demo stations or hands-on sessions.
- **Explain WHY for every recommendation.** "Agentforce session" alone is useless. "Agentforce Service workshop — directly maps to the $171K AF Contact Centre pilot at Stage 02. Customer attended 8 AF webinars in May. This is the in-person validation that could advance to BUILD TRUST." That's actionable.
- **Flag competitive opportunities.** If a competitor's product category has a Salesforce session that differentiates, recommend it: "Shield session — Trojan has Zendesk being displaced. Shield content validates the SF security/compliance story vs. Zendesk's basic offering."
- **Entitlement-based recommendations are gold.** "They own OmniStudio but aren't using it. This OmniStudio workshop could activate a dormant entitlement they're already paying for." That's free value for the customer.
- **Registration is the outcome.** Every recommendation should end with: who to register + how (AE action, or ask the customer directly). Without registration, the recommendation is academic.
- **Don't exclude ancillary events.** CxO dinners, SIC consultations, after-parties, and kiosk demos are all valid recommendations when they match the account's stage and stakeholder map.

## Integration with Other Skills

| Skill | What It Provides |
|---|---|
| **se-entitlement-decoder** | Product ownership → recommend deepening sessions for owned-but-unused capabilities |
| **se-competitive-intel-updater** | Competitor names → recommend sessions that differentiate against specific competitors |
| **se-deal-coach** | Methodology phase → recommend sessions that match advancement needs (discovery workshop for LISTEN, demo for BUILD TRUST, exec dinner for PARTNER) |
| **se-weekly-territory-review** | Territory-wide view → batch recommendations across all AEs at once |
| **se-account-research** | Full context → enrich recommendations with 5Cs, stakeholder map, and strategic priorities |

## Conversation Starters

- "What activities match Ecobee?"
- "Give me the full content menu for Stelpro"
- "Activity recommendations for Trojan Technologies"
- "What can I send or register Salford Group contacts for?"
- "What events and webinars match the Robinson pipeline?"
- "Deep dive on content options for Convergix"
