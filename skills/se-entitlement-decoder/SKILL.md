---
name: se-entitlement-decoder
description: "Decodes what a customer actually has. Pulls contracts (with expiry dates), line items (product SKUs), and resolves each SKU to its included entitlements via ProductLicenseMap. Answers: 'What can this customer actually do with what they've bought?' and 'What are they paying for that they're not using?'"
metadata:
  type: sales-operations
  version: "1.0"
---

# SE Entitlement Decoder

You answer the question every SE and customer asks: **"What does this customer actually have, and what can they do with it?"**

Salesforce SKU names are notoriously opaque. "Capped Salesforce Enterprise Subscription - Unlimited Edition" tells you nothing about what's included. This skill resolves SKU names → contract details → individual entitlements with descriptions, so anyone can understand what the customer has paid for, what they can use, and what they might not know they have.

## When to Use

- "What does [Account] actually own?"
- "What's included in Ecobee's contract?"
- "Decode the entitlements for [Account]"
- "What can [Account] do with what they've got?"
- "What are they paying for that they're not using?"
- Before any expansion conversation — know what they already have before pitching something new
- When a customer asks "what am I paying for?" or "can I do [X] with my current licenses?"

## Inputs

- **Account** (required) — account name, ID, or org62 URL
- **Focus** (optional) — specific product family or contract to decode

## Workflow

### Step 1: Pull Active Contracts

```
SELECT Id, ContractNumber, StartDate, EndDate, Status
FROM Contract
WHERE AccountId = '<account_id>' AND Status = 'Activated'
ORDER BY EndDate DESC
```

Output:

| Contract # | Start | End | Status | Days Until Expiry |
|---|---|---|---|---|
| [number] | [date] | [date] | Active | [X days] |

Flag contracts expiring within 6 months as :rotating_light:

### Step 2: Pull Won Opportunity Line Items (What They Bought)

```
SELECT Product2.Id, Product2.Name, Product2.Family, Product2.Description,
       Quantity, TotalPrice, Opportunity.CloseDate, Opportunity.Name
FROM OpportunityLineItem
WHERE Opportunity.AccountId = '<account_id>'
  AND Opportunity.IsClosed = true AND Opportunity.IsWon = true
ORDER BY Opportunity.CloseDate DESC
```

Deduplicate — show only the most recent purchase of each unique product.

### Step 3: Pull Asset Line Items (Current Active Entitlements)

```
SELECT Name, Apttus_Config2__ProductId__r.Name, Apttus_Config2__ProductId__r.Family,
       Apttus_Config2__Quantity__c, Apttus_Config2__StartDate__c, Apttus_Config2__EndDate__c,
       Apttus_Config2__AssetStatus__c
FROM Apttus_Config2__AssetLineItem__c
WHERE Apttus_Config2__AccountId__c = '<account_id>'
ORDER BY Apttus_Config2__EndDate__c DESC
```

### Step 4: Resolve SKUs to Entitlements (The Key Step)

For each unique product/SKU found in Steps 2-3, query the ProductLicenseMap:

```
SELECT LicenseDefinition.Name
FROM ProductLicenseMap
WHERE ProductId = '<product2_id>'
ORDER BY LicenseDefinition.Name
```

This returns the **actual licenses/entitlements included** in that SKU.

### Step 5: Categorize and Explain Entitlements

Group the entitlements into human-readable categories:

| Category | What It Means | Example Entitlements |
|---|---|---|
| **Core Platform** | Base CRM access and edition | EnterpriseEdition, CRM User AddOn, Offline User |
| **Sales** | Sales-specific capabilities | PipelineInspection, SalesforceForecastingAddOn, SalesCloudForecastCustomMeasure |
| **Service** | Service/support capabilities | Service Desk User, Entitlements Add On, CustomerServiceCatalogAddOn |
| **Field Service** | Mobile workforce management | FieldServiceAssetHierarchyAddOn, IndustriesFieldServiceAddOn |
| **Manufacturing** | Manufacturing Cloud features | Manufacturing Cloud Addon, ManufacturingFoundationAddOn, RebateMgmtUserAddOn, WarrantyLifecycleMgmtAddOn |
| **AI / Einstein** | AI capabilities included | Einstein Agent Basic AddOn, EinsteinBuilderFree, CallCoachingStandard, RecBuilderFree |
| **Data / Analytics** | Data and reporting | AnalyticsQueryServiceBase, DataProcessingEngine, DataCloudMetricsVisualizationAddOn |
| **Automation** | Flow, OmniStudio, BRE | FlowExecutionsUI50AddOn, OmniStudioDesignerAddon, BREDesignerAddOn, BRERuntimeAddOn |
| **Integration** | API and connectivity | API Request Limit 1000, ContextServiceBaseAddOn |
| **Marketing** | Marketing capabilities | Marketing User |
| **Content / Files** | Storage and content | File Storage 1 GB, Salesforce Content User |
| **Experience / Sites** | Portal and community | Flow Sites, Sites 24, SonicEmbeddedStoreBase |
| **Industry-Specific** | Vertical features | [varies by industry SKU] |

### Step 6: Identify Unused Entitlements (Whitespace-Within)

Compare entitled capabilities vs. what we know the customer is actually using (from cases, Slack, opportunity history):

| Entitlement | Status | Evidence |
|---|---|---|
| PipelineInspection | :white_check_mark: Likely in use | Sales Cloud deployed, pipeline opps exist |
| OmniStudioDesignerAddon | :x: Likely unused | No OmniStudio mentions in Slack or cases |
| Einstein Agent Basic | :large_yellow_circle: Unknown | Agentforce pilot in play — may not know they already have basic entitlement |
| Manufacturing Cloud Addon | :large_yellow_circle: Partially used | Mfg Cloud opp was purchased but case patterns suggest limited adoption |

**This is gold for expansion conversations.** "You're already paying for Einstein Agent Basic — did you know you can activate Agentforce capabilities without any new purchase?"

## Output Format

### Canvas Structure

Title: **[Customer] Product Map - dd/Mon/yy**

The canvas has three main sections in this order:

1. **Contracts + Line Items + Product License Maps** (the raw data — audit trail)
2. **Explanation: What This Means by Capability** (human-readable interpretation)
3. **What They Have But Might Not Know** (strategic insight for SE/AE)

---

### Section 1: Contracts → Line Items → Entitlements

This is the audit trail. Show the full hierarchy: Contract → Line Items → Product License Map.

```markdown
# :page_facing_up: Contracts & Entitlements

## Contract [#] | [Start] → [End] | [Annual Value] | [Status emoji]

| Line Item | Qty | Family | Product License Map (Entitlements Included) |
| --- | --- | --- | --- |
| [Product Name] | [X] | [Family] | [Comma-separated list of LicenseDefinition.Name values from ProductLicenseMap] |
| [Product Name] | [X] | [Family] | [Entitlements list] |
| [Product Name] | [X] | [Family] | [If no PLM records found: "No entitlement mapping found"] |

## Contract [#] | [Start] → [End] | [Annual Value] | [Status emoji]

| Line Item | Qty | Family | Product License Map (Entitlements Included) |
| --- | --- | --- | --- |
| [Product Name] | [X] | [Family] | [Entitlements] |
```

**Rules for Section 1:**
- One table per contract
- Every line item gets its own row
- The "Product License Map" column shows ALL entitlements from the ProductLicenseMap query for that product's Product2 ID
- If there are many entitlements (>10), summarize as: "96 entitlements — see breakdown in Section 2" and expand in the explanation section
- If ProductLicenseMap returns 0 records, write: "No entitlement mapping found — standalone product"
- Contracts ordered by expiry date (soonest first for urgency)
- Flag expiring contracts with :rotating_light: (within 6 months) or :red_circle: (within 30 days)

---

### Section 2: Explanation — What This Means by Capability

Group all entitlements across all contracts into human-readable categories. This is where the SE interprets what the customer can actually DO.

```markdown
# :unlock: What This Means (by Capability)

## :star: Core Platform
- [Edition] (X users)
- [User types included: CRM User, Offline, Mobile, etc.]

## :chart_with_upwards_trend: Sales
- [List each sales-related entitlement with a plain-English explanation]
- Pipeline Inspection ← "Can you see pipeline insights in your org?"

## :headphones: Service
- [List service entitlements]

## :wrench: Field Service
- [List FSL entitlements — flag if likely unused]

## :factory: Industry-Specific (Manufacturing / Financial / Healthcare)
- [List industry entitlements]

## :robot_face: AI / Einstein
- [List AI entitlements — HIGHLIGHT these, they're often unknown]

## :gear: Automation & Development
- [OmniStudio, BRE, Flows, DPE, etc.]

## :bar_chart: Data & Analytics
- [Analytics, Data Cloud entitlements]

## :globe_with_meridians: Integration & Platform
- [API limits, Sites, Storage]
```

---

### Section 3: Strategic Insights

```markdown
# :bulb: What They Have But Might Not Know

| Capability | Entitlement(s) | Opportunity |
| --- | --- | --- |
| [capability name] | [specific entitlements] | [positioning — how to bring this up with the customer] |

# :rotating_light: Contracts Expiring Soon

| Contract | Expires | Days | What's At Risk | Action |
| --- | --- | --- | --- | --- |
| [#] | [date] | [X] | [products on this contract] | [renewal/expansion action] |

# :brain: Key Strategic Insights
[Numbered list of 3-6 insights]
```

## Integration with Other Skills

| Skill | How Entitlement Decoder Feeds It |
|---|---|
| **se-competitive-intel-updater** | Confirms what Salesforce categories we're incumbent in (don't recommend what they already own) |
| **se-deal-coach** | "Customer already has Einstein Agent Basic — position Agentforce as activation, not new purchase" |
| **se-360-builder** | Footprint section gets precise product detail, not just "they have Sales Cloud" |
| **se-account-research** | Whitespace analysis becomes: "they own these entitlements but aren't using OmniStudio or Manufacturing for Service" |
| **se-agentforce-use-case-advisor** | Knows what AI entitlements exist before recommending additional SKUs |
| **promo-matcher** | Accurate current SKU data for promo eligibility |

## Rules

- **Always resolve SKUs to entitlements.** Never report just the product name. "Capped Enterprise Unlimited" means nothing. The 87 licenses inside it mean everything.
- **If a line item name is opaque, list it explicitly.** Product names like "Data Services Provisioning - Tableau Plus" or "Salesforce Foundations - Entitlements - Flex Credits" don't explain what they DO. When a product name isn't self-explanatory, include it in a "Line Items" sub-section under the contract with: Product Name | Quantity | Family | What It Likely Means. Add a "?" or "unclear" flag if even after research you can't determine what it provides. This lets the reader see exactly what's there and decide whether to investigate further.
- **Highlight the "didn't know they had it" items.** Einstein basics, OmniStudio, Manufacturing for Service — these are frequently entitled but never activated. That's the highest-value insight.
- **Flag expiring contracts proactively.** Within 6 months = renewal conversation. Within 3 months = urgent.
- **Connect to open opportunities.** If there's a $171K Agentforce opp open but they already have Einstein Agent Basic, that changes the positioning from "buy new" to "activate what you have + add the premium layer."
- **Don't assume usage from entitlement.** Having a license ≠ using it. Cross-reference with cases, Slack, and activity to assess actual adoption.
- **Deduplicate across contracts.** Same product appearing in multiple won opps = renewal/expansion history, not double-counting.
- **List opaque line items explicitly in the canvas.** For each contract, if ANY line item has a name that doesn't clearly communicate what it does, include a "Line Items" detail section below the contract summary table. Format:

```
### Contract [#] — Line Items

| Product Name | Qty | Family | What It Means |
|---|---|---|---|
| Manufacturing Cloud - Sales and Service - Unlimited Edition | 310 | Full CRM | Core platform: Sales + Service + Manufacturing Cloud features (96 entitlements — see breakdown below) |
| Agentforce for Service Add-on - Unlimited Edition | 168 | Service Cloud | AI-powered autonomous service agents — already purchased and active |
| Data Services Provisioning - Tableau Plus | 1 | Service Cloud | ? — unclear, likely enables data services for Tableau Plus tier |
| Salesforce Foundations - Entitlements - Flex Credits | 1 | Sales Cloud | Enables Flex Credit consumption against Foundations entitlement |
```

The "What It Means" column is the value-add. If you're unsure, say "? — unclear" rather than guessing. Better to flag uncertainty than provide incorrect interpretation.

## Conversation Starters

- "What does Ecobee actually own?"
- "Decode Stelpro's entitlements"
- "What's included in the Manufacturing Cloud SKU?"
- "What is Robinson paying for that they're not using?"
- "Does [Account] already have Einstein entitlements?"
- "What expires soon for [Account]?"
