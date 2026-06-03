---
name: se-account-workspace
description: "Manages persistent account project folders at ~/se-accounts/. Auto-creates account workspaces with CLAUDE.md context, opportunity sub-folders with deal history, and research archives. Triggered automatically when any SE skill runs on an account, or explicitly with 'create workspace for [Account]'."
metadata:
  type: sales-operations
  version: "1.0"
  depends_on: "se-account-research, se-deal-coach, se-360-builder"
---

# SE Account Workspace Manager

You manage persistent account project folders that store context, deal history, research outputs, and meeting notes for each account. Every account with active Stage 02+ opportunities gets its own workspace at `~/se-accounts/`.

## When to Use

- **Automatically:** Any time an SE skill runs on an account (deal coach, 360 builder, account research, territory review) — check if workspace exists, create if not, update if it does.
- **Explicitly:** "Create workspace for [Account]" or "Set up [Account] folder"
- **Maintenance:** "Update [Account] workspace" or "Refresh [Account] context"

## Folder Structure

```
~/se-accounts/
├── [account-slug]/                    ← kebab-case account name
│   ├── CLAUDE.md                      ← Account context file (auto-populated)
│   ├── opportunities/
│   │   ├── [opp-slug]/                ← kebab-case opp name
│   │   │   ├── deal-coach-YYYY-MM-DD.md    ← Dated snapshots
│   │   │   ├── deal-coach-YYYY-MM-DD.md    ← History builds over time
│   │   │   ├── scorecard.md                ← Current scorecard state
│   │   │   └── notes/                      ← Meeting notes for this opp
│   │   │       └── YYYY-MM-DD-[topic].md
│   │   └── [opp-slug]/
│   ├── research/
│   │   ├── 360-vision-YYYY-MM-DD.md        ← 360 Builder outputs (dated)
│   │   ├── case-insights-YYYY-MM-DD.md     ← Case analysis snapshots
│   │   ├── competitive-YYYY-MM-DD.md       ← Competitive intelligence
│   │   └── industry-research.md            ← Evergreen industry context
│   ├── deliverables/
│   │   ├── pov-YYYY-MM-DD.md              ← POVs created for customer
│   │   ├── challenge-statement.md          ← Current executive challenge
│   │   ├── demo-strategy-YYYY-MM-DD.md    ← Demo plans
│   │   └── exec-debrief-YYYY-MM-DD.md     ← Executive briefings
│   └── notes/
│       └── YYYY-MM-DD-[topic].md           ← General account notes
```

## CLAUDE.md Template (Auto-Populated)

When a workspace is created, generate this file from org62 + Slack + research:

```markdown
# [Account Name]

## Account Profile

- **Industry:** [X]
- **Location:** [City, Province, Country]
- **Employees:** [X]
- **Revenue:** [X if known]
- **Website:** [URL]
- **Org62 Account ID:** [ID]
- **AE:** [Name]
- **SE:** [Your Name]

## Salesforce Footprint

| Product | Licenses | Contract End |
|---|---|---|
| [product] | [count] | [date] |

**Total Contract Value:** $[X]
**Next Renewal:** [date]

## Stakeholder Map

| Name | Title | Role | Engagement |
|---|---|---|---|
| [name] | [title] | Champion/EB/TE/Blocker/Coach | Active/Passive |

## Open Opportunities

| Opportunity | Amount | Stage | Phase | Close | Health |
|---|---|---|---|---|---|
| [opp] | [$] | [##] | [LISTEN/BT/PARTNER] | [date] | [score/5] |

**Total Pipeline:** $[X]

## 5Cs Status

| C | Status | Evidence |
|---|---|---|
| C-level Priorities | [emoji] | [what we know] |
| Challenges | [emoji] | [what we know] |
| Competitive Threats | [emoji] | [what we know] |
| Compelling Events | [emoji] | [what we know] |
| Potential Crises | [emoji] | [what we know] |

## Key Context (Persists Across Conversations)

- [Key decision or history point that matters for future interactions]
- [E.g., "Customer rejected UE upgrade Dec 2025 — said 'no clear next steps'. Don't repeat that pattern."]
- [E.g., "Emerson building internal AI solution — competitive threat to Agentforce for Sales play"]
- [E.g., "IFS removal targeted EOY 2026 — hard compelling event for FSL"]

## Current Executive Challenge

[The executive challenge statement paragraph — updated when challenge builder runs]

## Last Updated

[Date] — [What was updated]
```

## Opportunity Folder Management

### Creating Opportunity Folders

When a deal coach or territory review identifies an active opp:

1. Create folder: `~/se-accounts/[account-slug]/opportunities/[opp-slug]/`
2. Opp slug format: `[product-shortname]-[amount-in-k]k` (e.g., `agentforce-155k`, `fsl-330k`, `service-cloud-132k`)
3. Create initial `scorecard.md` with current metrics

### Scorecard.md Format

```markdown
# SE Scorecard: [Opp Name]

**Opportunity ID:** [ID]
**Amount:** $[X] | **Stage:** [X] | **Close:** [Date]
**Last Updated:** [Date]

| Metric | Score | Comment |
|---|---|---|
| Process & Functional Discovery | [X] | [comment] |
| Compelling Event | [X] | [comment] |
| IT & Architecture Discovery | [X] | [comment] |
| Solutions Engagement Plan | [X] | [comment] |
| Unique Business Value | [X] | [comment] |
| Solution Fit | [X] | [comment] |
| Competitive Position | [X] | [comment] |
| Vision & Roadmap | [X] | [comment] |
| Exec Access & Sponsorship | [X] | [comment] |
| Implementation Strategy | [X] | [comment] |
| Partner Alignment | [X] | [comment] |

**Average:** [X.X]/5

## Score History

| Date | Average | Change | Reason |
|---|---|---|---|
| [date] | [score] | [Created/+X/-X] | [what changed] |
```

### Deal Coach History

Each time the deal coach runs, save the output as a dated snapshot:
- `deal-coach-2026-06-02.md`
- `deal-coach-2026-06-09.md`

This creates a timeline showing how the deal evolved — what was recommended, what was done (or not), and how the assessment changed.

## Auto-Trigger Rules

### When to Create a Workspace

Create a workspace automatically when:
1. **Any SE skill runs on an account** for the first time (deal coach, account research, 360 builder, case insights)
2. User explicitly asks: "create workspace for [Account]"

### When to Update CLAUDE.md

Update the account context when:
1. **Deal Coach runs** — update the opp table, 5Cs, scorecard
2. **360 Builder completes** — update everything (full refresh)
3. **Account Research runs** — update everything
4. **Territory Review surfaces new info** — update key context points
5. **User explicitly says** "update [Account] workspace"

### When to Add Opportunity History

Save a dated snapshot when:
1. **Deal Coach produces a canvas** — save the assessment to the opp folder
2. **Scorecard is created or updated** — update scorecard.md + add history row
3. **Meeting notes are processed** — save to the opp's notes/ folder

## Workspace Discovery

When any skill needs account context, check for an existing workspace:

```python
# Pseudo-logic
account_slug = slugify(account_name)  # "Stelpro Designs Inc." → "stelpro-designs"
workspace = f"~/se-accounts/{account_slug}"

if exists(workspace):
    read CLAUDE.md → pre-load context
    read opportunity scorecards → know current state
    read latest deal coach → know what was last recommended
else:
    create workspace
    populate from org62 + Slack
```

## Workspace Maintenance

### Weekly (Friday, as part of territory review):
- Update all account CLAUDE.md files for accounts that had activity this week
- Update scorecard.md for any scorecards that were touched/updated
- Flag workspaces with no activity in 30+ days

### Monthly:
- Archive opportunity folders for closed-won or closed-lost deals (move to `archived/`)
- Review key context points — remove stale ones, add new ones

### On AE Change:
- Update AE field in CLAUDE.md
- Add key context note: "[Date] — AE changed from [old] to [new]. Continuity risk."

## Integration with Skills

| Skill | Workspace Interaction |
|---|---|
| **se-deal-coach** | Reads opp history for context. Writes deal-coach-[date].md snapshot. Updates scorecard.md. |
| **se-360-builder** | Reads existing research (avoids re-researching). Writes to research/ folder. Updates CLAUDE.md fully. |
| **se-account-research** | Same as 360 builder. Master refresh. |
| **se-case-insight-analyzer** | Writes case-insights-[date].md to research/ folder. |
| **se-executive-challenge-builder** | Reads CLAUDE.md for context. Writes challenge-statement.md to deliverables/. Updates CLAUDE.md challenge section. |
| **se-demo-strategy** | Reads opp context + stakeholder map. Writes demo-strategy-[date].md to deliverables/. |
| **se-executive-debrief** | Reads everything. Writes exec-debrief-[date].md to deliverables/. |
| **se-weekly-territory-review** | Scans all active workspaces for health. Updates CLAUDE.md opp tables. |
| **se-opportunity-field-updater** | After updating org62, updates the scorecard.md and CLAUDE.md opp table. |

## Rules

- **Auto-create, never auto-delete.** Workspaces are created automatically but never removed without explicit user request.
- **CLAUDE.md is the source of truth** for persistent account context. If a conversation needs account history, read CLAUDE.md first.
- **Date everything.** Research, deal coaches, and deliverables are all dated. The timeline tells the story of the deal.
- **Don't duplicate org62.** CLAUDE.md stores context and interpretation, not raw CRM data. "Customer rejected UE in Dec 2025 — said no clear next steps" is valuable context. The opp stage and amount are in org62.
- **Scorecard history matters.** A score that went from 1.5 → 2.3 → 1.8 tells a different story than one that's been 2.3 forever. Track the trajectory.
- **Key Context section is human-curated.** Auto-populate from research, but the user should add/remove points that matter for future conversations. These are the "things I'd tell a colleague taking over this account."

## Conversation Starters

- "Create workspace for Stelpro"
- "Set up account folders for all of Philip's ≥$100K accounts"
- "Update the Laurentide workspace after today's call"
- "What's the deal history on the TICA-Smardt service opp?"
- "Show me the scorecard trajectory for Wakefield ERP"
