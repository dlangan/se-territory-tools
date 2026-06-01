# First-Run Setup

When you first use any SE territory skill, Claude will detect that configuration is missing and walk you through setup interactively. You'll be prompted for:

1. **Your name** and **Org62 User ID**
2. **Your manager's name** and **Org62 User ID** (used as default Tech_Exec__c)
3. **Your target org alias** (the SF CLI alias you use for org62)
4. **Your AE roster** (the AEs you support — used for territory reviews)

This configuration is saved to your local `.claude/projects/<project>/memory/` folder and persists across conversations.

## Finding Your User ID

In org62, navigate to your user record (Setup → Users → find yourself). The 18-character ID is in the URL:
```
https://org62.lightning.force.com/lightning/r/User/0053000000XXXXXXX/view
                                                   ^^^^^^^^^^^^^^^^^^
```

## Manual Setup (Optional)

If you prefer to set up manually, copy `memory-templates/user_se_config.md` to your memory folder:
```
cp memory-templates/user_se_config.md ~/.claude/projects/<your-project>/memory/user_se_config.md
```

Edit the file and fill in your details. Then add a pointer to your `MEMORY.md` index:
```
- [SE Config](user_se_config.md) — Manager ID, AE roster, org alias for territory tools
```

## Prerequisites

- **Salesforce CLI** (`sf`) installed and authenticated to org62
- **Claude Code** with Slack MCP plugin connected
- **Org62 access** with permissions to read/write Opportunity, Scorecard__c, and Scorecard_Data__c
