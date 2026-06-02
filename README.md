# SE Territory Tools

Claude Code skills for Solutions Engineering territory management, deal coaching, account research, and executive communication. Built for Salesforce SEs who manage multiple AE territories and need to keep pipeline intelligence, SE scorecards, and opportunity fields current across org62 and Slack.

## Quick Start — Which Skill Do I Use?

| I need to... | Use this skill |
|---|---|
| Update CRM after a meeting | `se-opportunity-field-updater` |
| Bulk update SE fields across a territory | `se-territory-bulk-updater` |
| Prep for Friday 1:1s with my AEs | `se-weekly-territory-review` |
| Coach a specific deal (methodology + next steps) | `se-deal-coach` |
| Prep for a meeting (full account picture) | `se-account-research` |
| Find hidden opportunities in support cases | `se-case-insight-analyzer` |
| Deep research a company (360 view + POV) | `se-360-builder` |
| Write the "why change now" executive narrative | `se-executive-challenge-builder` |
| Identify autonomous agent use cases | `se-agentforce-use-case-advisor` |
| Prep a demo (narrative, talk track, setup) | `se-demo-strategy` |
| Brief my RVP/VP (2-min strategic summary) | `se-executive-debrief` |
| Everything at once for one account | `se-account-research` (master orchestrator) |

## Skill Architecture

```
                    ┌─────────────────┐
                    │ Executive Debrief│  ← For leadership (2 min read)
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │ Account Research │  ← Master orchestrator
                    └────────┬────────┘
                             │
     ┌───────────────────────┼───────────────────────┐
     │                       │                       │
┌────┴─────┐  ┌──────────┐  ┌────┴─────┐  ┌────────┴───────┐
│360 Builder│  │Deal Coach│  │Case      │  │Demo Strategy   │
│(research) │  │(per-opp) │  │Insights  │  │(demo prep)     │
└─────┬─────┘  └────┬─────┘  └────┬─────┘  └────────────────┘
      │              │              │
      │     ┌────────┴───────┐     │
      │     │ Weekly Review  │     │
      │     │(territory scan)│     │
      │     └────────┬───────┘     │
      │              │              │
┌─────┴──────┐ ┌────┴─────┐ ┌─────┴──────┐
│ Challenge  │ │Agentforce│ │ Opp Field  │
│ Builder    │ │Use Cases │ │ Updater    │
└────────────┘ └──────────┘ └─────┬──────┘
                                  │
                           ┌──────┴──────┐
                           │Bulk Updater │
                           └─────────────┘
```

## Skills Included (11)

### Territory Management
| # | Skill | What It Does |
|---|---|---|
| 1 | **se-opportunity-field-updater** | Meeting notes → SE field updates on a single opp |
| 2 | **se-territory-bulk-updater** | Canvas/pipeline summary → bulk SE field updates across all opps for one AE |
| 3 | **se-weekly-territory-review** | Full weekly review: scorecards, methodology checks (5Cs + phase checklists), digital body language, slip risk → Slack canvas |

### Deal Coaching & Methodology
| # | Skill | What It Does |
|---|---|---|
| 4 | **se-deal-coach** | Single-deal deep dive: full Slack + Org62 sweep, methodology phase assessment, 5Cs, coaching recommendations → Slack canvas |
| 5 | **se-demo-strategy** | Builds tailored demo narrative: persona, scenario arc, product beats, talk track, technical setup → Slack canvas |

### Account Research & Strategy
| # | Skill | What It Does |
|---|---|---|
| 6 | **se-case-insight-analyzer** | Case history → opportunity signals. Severity trends, theme categorization, solution mapping |
| 7 | **se-360-builder** | Orchestrated account research: org62 + Slack + web research → frameworks (BMC, Porter's, Blue Ocean, SWOT) → Connected Vision |
| 8 | **se-agentforce-use-case-advisor** | Identifies autonomous agent use cases: industry → department → customer-specific Value Stream Mapping |
| 9 | **se-account-research** | Master orchestrator: coordinates all skills into a single comprehensive account strategy package |

### Executive Communication
| # | Skill | What It Does |
|---|---|---|
| 10 | **se-executive-challenge-builder** | Builds a 1-paragraph executive challenge statement framed through digital labour |
| 11 | **se-executive-debrief** | Creates a 2-minute RVP/VP debrief canvas: situation, evidence, risk, ask, timeline |

## Key Capabilities

- **Programmatic SE Scorecard creation** — creates parent + 11 weighted metrics directly in org62
- **Customer 360 Methodology integration** — LISTEN / BUILD TRUST / PARTNER phase assessments with checklists
- **5Cs Discovery framework** — C-level Priorities, Challenges, Competitive Threats, Compelling Events, Potential Crises
- **Digital labour framing** — autonomous agents, not chatbots. Work that gets done, not features that assist.
- **Slack canvas output** — dated, shareable, persistent artifacts for every major deliverable
- **Sub-agent orchestration** — 360 Builder and Account Research fan out parallel agents for speed

## Installation

1. Clone this repo
2. Copy the `skills/` folder into your `.claude/skills/` directory:
   ```bash
   cp -r skills/* ~/.claude/skills/
   ```
3. On first run, Claude will prompt you for configuration (manager ID, AE roster, etc.)
   - See [SETUP.md](SETUP.md) for details

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) CLI or Desktop
- Salesforce CLI (`sf`) authenticated to org62
- Slack MCP plugin connected
- Org62 permissions: read/write on Opportunity, Scorecard__c, Scorecard_Data__c, Case, Contract

## Salesforce Fiscal Calendar

| Quarter | Months |
|---------|--------|
| FQ1 | Feb – Apr |
| FQ2 | May – Jul |
| FQ3 | Aug – Oct |
| FQ4 | Nov – Jan |

## How It Works

```
Meeting Notes / Canvas / "research [Account]" / "coach me on [Deal]"
        │
        ▼
┌─────────────────────────────┐
│  1. Query Org62             │
│  2. Search Slack            │
│  3. SE Scorecard Check      │
│  4. Methodology Assessment  │
│  5. External Research       │
│  6. Synthesis & Coaching    │
└───────────────┬─────────────┘
                │
                ▼
┌─────────────────────────────┐
│  Outputs:                   │
│  • Slack Canvas (dated)     │
│  • SE Fields → Org62       │
│  • Scorecards → Org62      │
│  • Actions (time-bound)    │
└─────────────────────────────┘
```

## Also Works With (Local Skills Not in This Repo)

These skills exist locally and feed into the territory tools system:

| Skill | What It Does | Integration |
|---|---|---|
| `call-notes-analyzer` | Structures meeting transcripts into discovery frameworks | Feeds → `se-opportunity-field-updater` |
| `pipeline-finder` | Territory whitespace analysis | Feeds → `se-weekly-territory-review` |
| `promo-matcher` | Identifies eligible promotions for deals | Feeds → `se-deal-coach` |
| `sf-dse` | DSE-level strategic review of deliverables | Reviews → any canvas output |

## License

MIT
