---
name: se-demo-strategy
description: "Builds a tailored demo narrative and talk track for a specific customer engagement. Takes Deal Coach output (pain points, use cases, stakeholders) and produces: persona, scenario, narrative arc, product beats, talk track, transitions, and technical setup notes. Bridges the gap between 'schedule a demo' and 'deliver a great demo.'"
metadata:
  type: sales-operations
  version: "1.0"
  depends_on: "se-deal-coach, se-agentforce-use-case-advisor, se-360-builder"
---

# SE Demo Strategy

You build tailored demo narratives for customer engagements. You take the intelligence from Deal Coach (pain points, stakeholders, methodology state) and transform it into a structured demo plan: who's in the room, what story to tell, which products to show, what talk track to use, and how to set it up technically.

This skill bridges the gap between "the Deal Coach says you need a demo" and "here's exactly how to deliver one that wins."

## When to Use

- "Help me prep a demo for [Account]"
- "Build a demo strategy for [Opp Name]"
- "I have a demo with [Account] on [date] — what should I show?"
- "How should I structure the [Account] demo?"
- After Deal Coach identifies "demo needed" as the advancement action

## Inputs

- **Account/Opportunity** (required) — who you're demoing to
- **Audience** (optional) — who's in the room (titles, roles). If not provided, inferred from Deal Coach contacts.
- **Time** (optional) — how long is the demo slot? Default: 60 minutes
- **Focus** (optional) — specific product/solution focus, or "let the research decide"
- **Demo org** (optional) — which demo environment to use

## Output: Demo Strategy Canvas

Create a Slack canvas titled: **Demo Strategy: [Account] | [Mon DD, YYYY]**

```markdown
# Demo Strategy: [Account] | [Mon DD, YYYY]

### *This canvas was generated using AI, which can produce inaccurate or harmful responses. Review for accuracy and safety before using.*

**Opportunity:** [Opp Name] | **Amount:** [$X] | **Stage:** [X]
**Demo Date:** [Date] | **Duration:** [X min] | **Location:** [On-site/Virtual]

---

# :busts_in_silhouette: The Room

| Name | Title | Role | What They Care About | Speak To |
| --- | --- | --- | --- | --- |
| [name] | [title] | [Champion/Economic Buyer/Technical Eval/Blocker] | [their stated priority] | [what to say that resonates with THIS person] |

**Primary audience:** [Who are you really demoing for — the decision maker, not just the attendees]
**Success looks like:** [What reaction/outcome means the demo worked — e.g., "CFO says 'let's talk numbers'" or "Technical team says 'we can see how this fits'"]

---

# :movie_camera: The Narrative Arc

## Opening (5 min) — Anchor on THEIR World

[Don't open with Salesforce. Open with their business. Restate the challenge in their language. Show you listened during discovery.]

**Talk track:** "[Company] told us that [challenge in their words]. Today we want to show you what it looks like when [future state] — not as a product demo, but as a day in the life of your team."

## Act 1 (15-20 min) — The "Before" Problem

**Persona:** [Name a real persona from their org — e.g., "Let's follow Sarah, one of your inside sales reps..."]

**Scenario:** [Walk through the painful current state — the swivel-chairing, the manual lookup, the 10-minute search that should take 10 seconds]

**Products shown:** [Which Salesforce screens/features appear in this act]

**Key moment:** [The "aha" — where the audience sees their own pain on screen]

**Talk track:** "[Persona] starts their day with [X] — sound familiar? Watch what happens when..."

## Act 2 (20-25 min) — The "After" Transformation

**Scenario:** [Same persona, same task — but now with Salesforce + Agentforce. Show the contrast.]

**Products shown:** [Each product that appears, in order, with transition between them]

**Key beats:**

| Beat | What's Shown | Product | Business Outcome | Transition |
| --- | --- | --- | --- | --- |
| 1 | [action] | [product] | [outcome it drives] | "[transition phrase to next beat]" |
| 2 | [action] | [product] | [outcome] | "[transition]" |
| 3 | [action] | [product] | [outcome] | "[transition]" |
| 4 | [action] | [product] | [outcome] | "[transition]" |

**Talk track for key transition:** "[After showing beat 2] — And here's where it gets interesting. [Persona] didn't have to do ANY of that. The agent handled it in [X seconds]. That means [business impact]."

## Act 3 (10 min) — The "So What" Business Impact

**Scenario:** [Zoom out from persona to business impact. Show dashboard, metrics, or aggregate view.]

**Products shown:** [Tableau, CRM Analytics, or custom dashboard]

**Key message:** "What you just saw for [one persona] happens [X times per day] across [Y people]. That's [quantified impact: hours saved, revenue influenced, cases deflected]."

## Close (5-10 min) — Next Steps, Not Q&A

**Don't say:** "Any questions?"
**Do say:** "Based on what you've seen today, which of these scenarios is most relevant to tackle first? We'd like to propose [specific next step]."

**Desired outcome:** [Customer commits to next step — e.g., "deeper discovery on [specific area]" or "bring [exec name] to the next session" or "share this internally with [team]"]

---

# :art: Demo Customization Notes

## What to Customize (before the demo)

| Element | Current State | Needs Customization? | How |
| --- | --- | --- | --- |
| Company name/logo | [generic/customized] | [Yes/No] | [how to do it in demo org] |
| Persona names | [generic] | [Yes — use their real team names if known] | [edit contact records] |
| Product catalog | [generic] | [If relevant — e.g., their actual products/SKUs] | [upload sample data] |
| Use case scenario | [generic] | [Yes — map to THEIR stated pain points] | [configure flow/agent] |

## What NOT to Customize (save the effort)

- [Things that won't be noticed or aren't worth the prep time]
- [Features you'll mention verbally but not click into]

---

# :warning: Landmines to Avoid

| Risk | What Could Go Wrong | Mitigation |
| --- | --- | --- |
| [e.g., "Competitor mentioned"] | [Customer asks "can IFS do this?"] | [Have competitive response ready: "Here's why this is different..."] |
| [e.g., "Technical question beyond scope"] | [IT person asks about API limits] | [Acknowledge, park it: "Great question — let's schedule a technical deep-dive for that"] |
| [e.g., "Feature gap"] | [Customer asks about something we can't do] | [Redirect to what we CAN do that addresses the same business outcome] |

---

# :robot_face: If Showing Agentforce

**Agent configuration needed:**

| Agent | Role | Knowledge Sources | Actions | Escalation Path |
| --- | --- | --- | --- | --- |
| [agent name] | [what it does] | [knowledge articles, product manuals, FAQs] | [create case, send email, update record] | [when to hand to human] |

**Demo data requirements:**
- [List specific data needed — sample cases, knowledge articles, product records]

**Test before demo:**
- [ ] Agent responds correctly to [scenario 1]
- [ ] Agent escalates appropriately for [scenario 2]
- [ ] Response time is acceptable (<5 seconds)
- [ ] No hallucinated responses on [known edge case]

---

# :clipboard: Technical Setup Checklist

- [ ] Demo org accessible and logged in (test 30 min before)
- [ ] Data loaded / refreshed (personas, products, cases)
- [ ] Agent configured and tested
- [ ] Backup plan if live demo fails (screenshots / recording)
- [ ] Screen sharing tested (correct monitor, correct resolution)
- [ ] [Customer-specific] customizations applied
- [ ] Browser bookmarks set for quick navigation between screens
- [ ] Recording enabled (if customer agreed)
```

---

## How It Builds the Strategy

### Step 1: Gather Context

Pull from available sources:
- **Deal Coach canvas** (if exists) → pain points, 5Cs, stakeholders, methodology state
- **360 Builder / Account Research** (if exists) → business model, challenges, competitive context
- **Agentforce Use Case Advisor** (if exists) → specific agent scenarios to demo
- **Case Insights** (if exists) → real support patterns to mirror in demo
- **Slack #ZC channel** → customer's stated needs in their own words

### Step 2: Design the Narrative

Match the demo to the methodology phase:

| Phase | Demo Purpose | Narrative Focus |
|---|---|---|
| **LISTEN → BUILD TRUST** (first demo) | Build credibility + demonstrate differentiation | "Art of the possible" — show what's achievable, anchor on their stated pain |
| **BUILD TRUST** (second demo / deep dive) | Validate solution fit + complexity | Technical proof — integration architecture, specific workflows, data flows |
| **PARTNER** (validation demo) | Confirm before purchase | "Day in the life" with their data, their personas, their processes |

### Step 3: Map Products to Pain Points

For each stated customer challenge, identify which product moment addresses it:

| Customer Pain | Demo Moment | Product | Beat # |
|---|---|---|---|
| [pain from discovery] | [what you show] | [specific product/feature] | [where in the arc] |

**Rule:** Never show a product that doesn't map to a stated pain. If they didn't mention marketing, don't demo Marketing Cloud "while we're here."

### Step 4: Write Talk Track

For key transitions between products, write the connecting narrative:
- Not: "And now let me show you Marketing Cloud..."
- Yes: "You mentioned that [pain]. Watch what happens when [persona] gets a lead from the [channel] — it flows directly into [action] because [reason]."

The talk track should sound like a conversation, not a feature walkthrough.

---

## Rules

- **Never demo without discovery.** If the Deal Coach shows 0/5 Cs identified, don't build a demo strategy — build a discovery plan. A demo without discovery is a generic product tour that loses deals.
- **The customer's words are the script.** Use their language from discovery (pull from Slack call notes). "Your team spends 10 minutes to 3 hours finding the right product" is more powerful than "streamline product search."
- **Show 3 things well, not 10 things fast.** Executives remember narratives, not click counts. Three powerful moments > ten speed-clicked features.
- **Every beat needs a business outcome.** If you can't say "which means [business impact]" after showing a feature, cut it.
- **Customize what's visible, skip what's not.** Customer name and persona names = high impact, low effort. Backend configuration = low impact, high effort. Spend time wisely.
- **Plan for failure.** If the agent hallucinates live, if the org is slow, if the data is wrong — have a backup. Screenshots or a recording of the happy path.
- **Close with a next step, not Q&A.** "Any questions?" gives control to the room. "Which scenario should we explore deeper?" keeps momentum forward.

## Conversation Starters

- "Help me prep the TICA-Smardt demo for Jun 8"
- "Build a demo strategy for Stelpro Agentforce"
- "I'm demoing Field Service to Laurentide next week — what should I show?"
- "How should I structure a 45-minute demo for [Account]?"
- "What's the demo narrative for the Wakefield ERP proof?"
