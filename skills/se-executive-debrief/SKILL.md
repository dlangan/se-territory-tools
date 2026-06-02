---
name: se-executive-debrief
description: "Creates an executive-ready account debrief canvas designed for RVP/VP-level consumption. Distills the full account research into a 1-page strategic summary: situation, opportunity, risk, ask. No methodology jargon — pure business narrative with a clear recommendation."
metadata:
  type: sales-operations
  version: "1.0"
  depends_on: "se-account-research, se-deal-coach, se-executive-challenge-builder"
---

# SE Executive Debrief

You create a concise, executive-ready account debrief designed for RVP/VP/SVP consumption. This is NOT the full research package — it's the distilled strategic summary that answers: "Should I invest executive attention here, and what specifically do you need from me?"

## When to Use

- "Create an executive debrief for [Account]"
- "I need to brief Kat on [Account]"
- "Prep the RVP summary for [Company]"
- "Executive summary for [Account] — one page"
- After completing an account research run — to produce the leadership artifact
- Before asking for executive sponsorship, resource allocation, or escalation

## Inputs

- **Account** (required) — account name, ID, or "based on the research we just did"
- **Ask** (optional) — what you want from the executive ("exec sponsor", "resource allocation", "deal review", "escalation")
- **Context** (optional) — prior account research canvas, Deal Coach outputs, or conversation context

## The Executive's Questions (answer all of them)

Every RVP/VP reads an account brief asking exactly 5 questions:

1. **"Why should I care?"** — What's the opportunity size and strategic value?
2. **"Is it real?"** — What's the evidence that this will close?
3. **"What's the risk?"** — What could kill it?
4. **"What do you need from me?"** — What's the specific ask?
5. **"What's the timeline?"** — When does this need to happen?

The debrief answers all 5 in under 2 minutes of reading time.

## Canvas Output

Create a Slack canvas titled: **Executive Debrief: [Company] | [Mon DD, YYYY]**

```markdown
# Executive Debrief: [Company] | [Mon DD, YYYY]

**Prepared for:** [RVP/VP Name] | **Prepared by:** [SE Name] | **AE:** [AE Name]

---

# :dart: The Opportunity

**Account:** [Company Name] | **Industry:** [X] | **Location:** [X]
**Total Pipeline:** $[X] across [Y] opps | **Realistic Close (this FQ):** $[X]
**Strategic Value:** [1 sentence — why this account matters beyond the immediate $]

---

# :page_facing_up: Situation (30 seconds)

[3-4 sentences max. What's happening with this account RIGHT NOW. Not history — current state. Include: what they own, what's in play, what just changed (a meeting, a signal, a risk). Written for someone who has never heard of this account before.]

---

# :bulb: The Challenge

[The executive challenge statement — single paragraph from se-executive-challenge-builder. Digital labour framed. This is what we'd say to THEIR CEO.]

---

# :white_check_mark: Evidence It's Real

| Signal | Evidence | Date |
| --- | --- | --- |
| [Customer pull signal] | [Specific quote, action, or commitment from customer] | [Date] |
| [Executive engagement] | [Who's involved at what level] | [Date] |
| [Methodology progress] | [Phase + 5Cs + checklist completion] | [Date] |
| [Partner/ecosystem] | [SI engaged, competitor displaced, etc.] | [Date] |

**Methodology Score:** [X/5 Cs identified] | [Phase: LISTEN/BUILD TRUST/PARTNER] | [Scorecard: X.X/5]
**Confidence:** [High/Medium/Low] — [1 sentence justification]

---

# :warning: What Could Kill It

| Risk | Probability | Impact | Mitigation |
| --- | --- | --- | --- |
| [Risk 1 — e.g., "NetSuite selected for ERP"] | [High/Med/Low] | [$ at risk] | [What we're doing about it] |
| [Risk 2 — e.g., "Champion leaves role"] | [Probability] | [Impact] | [Mitigation] |
| [Risk 3 — e.g., "Budget freeze Q3"] | [Probability] | [Impact] | [Mitigation] |

**Slip Risk:** [X]% | **Primary Vector:** [Inactivity/Political/Commercial/Procurement/Competitive/Technical]

---

# :raised_hand: The Ask

**What I need:** [Be specific — one sentence]

[Expand in 2-3 sentences max. Examples:]
- "Executive sponsor introduction: I need you to connect with [their VP Name] to validate the platform vision at the presidential level. Philip has champion-level access (William, Dir CX) but the $668K cluster requires executive-to-executive alignment with Louis Beaulieu (President)."
- "Resource allocation: I need a specialist SE (Field Service) assigned to this account for the Jun 8 demo. Emilie is covering but this is a competitive eval against IFS — we need depth."
- "Deal review: This $330K deal has been at Stage 02 for 175 days despite re-engagement signals. I need your coaching on whether Sep 30 is achievable or if we should reforecast to Dec."
- "Air cover: The AE hasn't followed up in 117 days on a $286K opp. I've flagged it in the territory review but need your support in the coaching conversation."

---

# :calendar: Timeline

| Milestone | Date | Owner | Status |
| --- | --- | --- | --- |
| [Next critical event — e.g., "Customer readback session"] | [Date] | [Who] | [Scheduled/Pending/At Risk] |
| [Following event — e.g., "Architecture response due"] | [Date] | [Who] | [Status] |
| [Close target] | [Date] | [Who] | [Depends on: X] |

**Decision point:** [When does the exec need to act by for it to matter? — e.g., "If exec intro doesn't happen by Jun 12, the Jul 31 renewal window closes without strategic alignment."]

---

# :brain: Context (Read Only If Interested)

## Account Cluster

| Opportunity | Amount | Stage | Health | Close |
| --- | --- | --- | --- | --- |
| [opp] | [$] | [##] | [score/5] | [date] |

**Full Account Research:** [Link to master research canvas if it exists]
**Deal Coach:** [Link(s) to individual deal coaching canvases]

## Digital Labour Opportunity

[2-3 sentences: what autonomous agents could do for this customer, quantified. This gives the exec the "innovation story" to tell their leadership.]

## Competitive Context

[1-2 sentences: who we're competing against and our differentiation. Only include if competition is active.]
```

---

## Tone & Register

- **No methodology jargon.** Don't say "5Cs" or "BUILD TRUST phase" or "SE Scorecard." Say "we've completed discovery" or "customer has confirmed us as vendor of choice" or "deal health is strong."
- **Business language only.** Revenue, pipeline, close dates, risks, asks. Not frameworks.
- **Assume they have 2 minutes.** If they read the TL;DR (Situation + The Ask), they should know whether to keep reading.
- **The Ask is the point.** Everything else exists to justify The Ask. If you don't have an ask, you don't need a debrief — you need a territory review.
- **Be direct about risk.** Executives respect honesty. "This deal could die if [X] happens" is more valuable than "everything is progressing well."

## When NOT to Create This

- If there's no ask. An executive debrief without an ask is just an FYI — send a Slack message instead.
- If the deal is <$50K and routine. Don't waste executive attention on small-ball.
- If the account research isn't done. Don't create a debrief from thin air — run the research first, then distill.

## Integration with Other Skills

| Skill | What It Provides to the Debrief |
|---|---|
| **se-account-research** | Full context — distill from the master canvas |
| **se-deal-coach** | Methodology score, confidence level, evidence signals |
| **se-executive-challenge-builder** | The challenge paragraph (used verbatim) |
| **se-weekly-territory-review** | Territory-level context if the exec asks "how does this fit the bigger picture?" |
| **se-agentforce-use-case-advisor** | The digital labour opportunity summary |

## Rules

- **Under 2 minutes read time.** If an RVP can't absorb this in 2 minutes, it's too long. Cut.
- **One ask per debrief.** Multiple asks dilute urgency. Pick the highest-leverage one.
- **Evidence table is mandatory.** Executives don't trust "it's going well." They trust: "Customer said X on [date]. VP attended the demo. Partner is co-investing."
- **Timeline must include a decision point.** When does the exec need to act for it to matter? Without this, there's no urgency to respond.
- **Link to detail, don't include it.** The full research canvas exists for depth. The debrief exists for decision-making.
- **Quantify everything possible.** "$668K cluster" not "large account." "5/5 Cs identified" translates to "discovery is complete." "40% slip risk" not "some risk."

## Conversation Starters

- "Create an executive debrief for Stelpro — I need an exec sponsor ask"
- "Brief Kat on the TICA-Smardt situation"
- "I need an RVP summary for Wakefield — the ask is deal review"
- "Prep a debrief for Laurentide — I need a specialist SE allocated"
- "Executive summary for Mcdougall — I need air cover for the coaching conversation with Frank"
