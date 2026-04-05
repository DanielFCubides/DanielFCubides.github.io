---
title: "What Stakeholders Really Want to Know — And Why Most Updates Miss It"
date: 2026-04-05T11:00:00-05:00
draft: false
featured_image: /images/ideas.jpg
tags: ["engineering-management", "communication", "leadership", "stakeholders"]
categories: ["engineering-management"]
---

# What Stakeholders Really Want to Know — And Why Most Updates Miss It

*One format sent to everyone answers no one's question well.*

---

Here is a mistake I see engineering managers make constantly — and one I made myself for years.

They write one update. They send it to everyone.

Their VP gets the same message as their direct manager. Their peer team lead gets the same as their own engineers. One format, one level of detail, one angle — broadcast to an entire distribution list and checked off the mental to-do.

The problem is not the effort. The problem is that each of those people is asking a fundamentally different question. And when one message tries to answer all of them, it usually answers none of them well.

---

## The Four Audiences Every Engineering Manager Has

Most managers communicate upward, downward, and sideways — but treat all three directions as variations of the same task. They are not. Each direction contains a different audience with a different relationship to your work, a different decision-making context, and a different definition of "useful information."

---

### Audience 1: Executives and Skip-Level Leaders

**The question they are silently asking:** *Should I be worried?*

Executives operate at altitude. They are tracking multiple initiatives simultaneously, managing their own stakeholders, and making resource allocation decisions across the organization. They do not need — and often cannot use — a detailed understanding of your team's technical work.

What they need is a **confidence signal** with enough context to defend it upward.

Their three core questions, in order:

1. **Are we on track?** Is the project proceeding as expected?
2. **Are there risks I should know about?** Is there anything that might become a problem before we next speak?
3. **Is the team healthy?** Is there capacity, morale, or retention risk I should factor into decisions?

Notice what is *not* on this list: what the team worked on this week, which tickets were completed, what technical decisions were made. Those are noise to an executive unless they directly connect to one of the three questions above.

```
Executive Update Template:

STATUS:    [On Track / At Risk / Blocked] for [date/milestone]
KEY FACTS: • [1-2 sentences on progress, with numbers]
           • [Next milestone]
RISKS:     [If any: what, impact, mitigation — 1 sentence each]
NEEDS:     [If any: explicit ask with deadline]
```

The entire update should be readable in under 90 seconds. If it takes longer, you have included information that belongs to a different audience.

---

### Audience 2: Your Direct Manager

**The question they are silently asking:** *Do you need anything from me?*

Your direct manager has more context than your skip-level but a different accountability. Their job is to unblock you, support your development, and translate your team's work into organizational context. They are not primarily worried about whether to worry — they are worried about whether they are being a good manager to you.

Their three core questions:

1. **What is the team working on?** Do they have enough context to support you and represent your team accurately?
2. **Are there blockers?** Is there anything they should be removing — a dependency, a decision, a resource gap?
3. **Do you need support?** Are you okay? Is the team okay?

This is the right venue for early-stage risks, team dynamics concerns, and genuine uncertainty. Executives need conclusions. Your direct manager can handle process.

```
Direct Manager Update:

FOCUS:        What the team is primarily working on
PROGRESS:     What moved forward (with brief context)
BLOCKERS:     What is slowing us down or at risk (with explicit ask if any)
TEAM HEALTH:  One honest sentence on energy, morale, or concerns
AHEAD:        What comes next and any decisions needed
```

---

### Audience 3: Peer Teams

**The question they are silently asking:** *Does anything you're doing affect my work?*

Peer teams — other engineering teams, product, design, data, operations — have a specific and often underserved information need. They do not need to understand your team's full work. They need to understand the **intersection** of your work and theirs.

Their three core questions:

1. **What dependencies exist?** Are they waiting on something from your team? Are you waiting on something from them?
2. **What should we know that affects our work?** Are you making a decision, changing a timeline, or shipping something that creates ripples on their side?
3. **Is anything changing?** Scope changes, timeline shifts, technical decisions with cross-team implications.

Peer team updates are the most underinvested communication in most engineering organizations. They tend to happen reactively — someone discovers a dependency conflict in a meeting and then scrambles to align. The cost of that pattern in rework and eroded cross-team trust is enormous.

```
Peer Team Update:

SHIPPING THIS WEEK:  What we're releasing that might affect you
WAITING ON:          Open dependencies we have on your team
HEADS UP:            Upcoming decisions or changes with cross-team impact
QUESTIONS:           Anything we need input on from your side
```

---

### Audience 4: Your Own Team

**The question they are silently asking:** *Do I know enough to do my best work?*

This is the audience most managers forget. Information should flow *from* the team, not *to* it, or so the thinking goes. But your engineers — regardless of seniority — need direction, context, and clarity to operate at their best.

Their three core questions:

1. **What is the direction?** Where are we going? Why? What does success look like?
2. **What decisions have been made?** What is settled so they can stop debating and start building?
3. **What is expected of me?** What does good look like for their specific work?

An engineering manager who communicates well downward creates a team that makes better autonomous decisions — because they have the context to do so. This is what enables real delegation.

```
Team Update (5-15 Format):

WINS:       What the team accomplished this week (celebrate this)
FOCUS:      What the priority is next week and why
DECISIONS:  What is settled — so we stop relitigating it
CONTEXT:    Relevant org news that affects our work
OPEN:       What is still being figured out (be honest about this)
```

---

## The Audience Matrix

| Audience | Core Question | Key Needs | Format | Frequency |
|---|---|---|---|---|
| Executive / Skip-level | Should I be worried? | On track? Risks? Health? | 3-5 bullet BLUF | Weekly or bi-weekly |
| Direct Manager | Do you need anything? | Work focus, blockers, health | 5-15 style, candid | Weekly |
| Peer Teams | Does this affect my work? | Dependencies, changes | Short digest | Weekly or bi-weekly |
| Your Team | Do I know enough? | Direction, decisions, expectations | 5-15 + context | Weekly |

If you are sending the same update to all four, you are optimizing for none of them.

---

## What Happens When You Forward the Wrong Update

I want to be concrete about what goes wrong when you send a direct-manager update to a VP.

Imagine you write a weekly update for your direct manager — operational, candid, covering a blocker with the auth service, a team member who is onboarding, and a risk to next sprint's delivery. Thorough and well-written.

Then you forward it to your VP.

Your VP reads: "We had a blocker with the auth service." They do not know if this is resolved or escalating. They read: "Priya is onboarding." They do not know if this affects delivery timelines. They read: "There is a risk to next sprint's delivery." Now they are worried — but they have no idea of the magnitude, the mitigation, or whether they need to act.

You have not informed your VP. You have alarmed them with context they cannot interpret.

This is a very common pattern. It almost always triggers the exact micromanagement and excessive follow-up that the update was meant to prevent.

> **The rule:** Never forward an update written for one audience to a different audience. Rewrite it. It takes five minutes and prevents hours of confusion.

---

## Building Your Stakeholder Map

The practical output of this session is a **Stakeholder Map** — a document that describes each of your key stakeholders, their information needs, their preferred format, and their preferred frequency.

```
STAKEHOLDER MAP

Name / Role:        [Name], [Title]
Relationship:       [Direct manager / Skip-level / Peer / Report]
Core Question:      [The one question they most need answered]
Key Info Needs:     [2-3 specific things they need regularly]
Preferred Format:   [Written async / Verbal / Dashboard / Bullets]
Preferred Channel:  [Slack / Email / Notion / 1:1 meeting]
Frequency:          [Weekly / Bi-weekly / Monthly]
Current Gap:        [What are they not getting that they need?]
Trust Level:        [High / Medium / Low]
Uncertainty Level:  [High / Medium / Low]
```

Fill this out for every significant stakeholder — typically 4 to 8 people. Be honest about the gaps. The gap column is where your communication improvement opportunities live.

---

## The Exercise

Set aside 30 minutes to complete your Stakeholder Map.

1. List every significant stakeholder — manager, skip-level, peer leads, key product partners, your team as a group.
2. For each one, fill in the template. Be specific about their core question and honest about the current gap.
3. Ask: **am I currently sending updates that answer each stakeholder's core question?**
4. Identify the one stakeholder whose information needs are most underserved. That is your highest-priority communication improvement.

Keep this map somewhere accessible. Revisit it quarterly — stakeholders change roles, priorities shift, trust levels evolve.

---

*Next in this series: [the measurable cost of a broken status update system — in time, attention, trust, and team culture](#).*

*Tags: engineering-management · stakeholders · communication · leadership · distributed-teams*
