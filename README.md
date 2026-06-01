# SE Territory Tools

Claude Code skills for Solutions Engineering territory management. Built for Salesforce SEs who manage multiple AE territories and need to keep pipeline intelligence, SE scorecards, and opportunity fields current across org62 and Slack.

## Skills Included

### 1. SE Opportunity Field Updater
**Trigger:** Paste meeting notes + opp link, or say "update SE fields"

Reads meeting notes (Google Docs, Slack canvases, pasted text) and generates structured SE field updates. Can push directly to org62.

Fields managed: SE Engagement, Product Fit, Functional/Technical Selection, Solution Description, SE Next Steps, SE Comments, Tech Exec.

### 2. SE Territory Bulk Updater
**Trigger:** "Bulk update from this canvas" or "update SE fields across [AE]'s territory"

Takes a territory review canvas (or similar pipeline summary), identifies all active opportunities, searches Slack for per-account evidence, synthesizes SE field updates, and pushes to org62 in bulk.

### 3. SE Weekly Territory Review
**Trigger:** "Run a territory review for [AE Name]"

The full weekly review workflow:
- SE Scorecard checks on all opps (create/touch/update)
- Digital body language health check (org62 activity + Slack signals)
- Slip probability scoring per opportunity
- Forecast reality check
- Publishes a formatted territory review canvas to Slack

**Scorecard management:**
- Creates new SE Scorecards programmatically (parent + 11 metrics with weighted scores)
- Updates existing scorecards (specific metric changes with evidence comments)
- Touches unchanged scorecards (refreshes LastModifiedDate to signal active monitoring)

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
- Org62 permissions: read/write on Opportunity, Scorecard__c, Scorecard_Data__c

## Configuration

On first use, you'll be prompted interactively for:
- Your Org62 User ID
- Your manager's User ID (default Tech Exec)
- Your AE roster (names)
- Your SF CLI org alias

Config is stored in Claude's memory system and persists across conversations.

## Salesforce Fiscal Calendar

| Quarter | Months |
|---------|--------|
| FQ1 | Feb – Apr |
| FQ2 | May – Jul |
| FQ3 | Aug – Oct |
| FQ4 | Nov – Jan |

## How It Works

```
Meeting Notes / Canvas / "review for [AE]"
        │
        ▼
┌─────────────────────────┐
│  1. Query Org62 Opps    │
│  2. Search Slack        │
│  3. SE Scorecard Check  │
│  4. Health Assessment   │
│  5. Slip Analysis       │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Push to Org62:         │
│  • SE Fields            │
│  • Scorecards           │
│  • Territory Canvas     │
└─────────────────────────┘
```

## License

MIT
