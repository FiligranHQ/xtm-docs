---
title: Rehearse a cyber crisis with the people who would live it
description: Play an intrusion again to your crisis cell in OpenAEV, give it the cost of the breach from OpenCRQ and a briefing from an XTM One agent, then collect its lessons learned.
doc_type: journey
journey: rehearse-a-crisis
products: [openaev, opencrq, xtm-one]
personas: [exercise-planner, risk-manager, ai-builder]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 64
---

At the end of this journey, your crisis cell has lived the night of an intrusion again: the alert of the endpoint detection and response (EDR) tool, the call to the bridge, the question it must answer and the press article that piles on. It knows what the breach would cost in money and has a briefing to decide from, and you have its lessons learned. The exercise planner drives it in OpenAEV; OpenCRQ prices the breach, and an XTM One AI agent briefs the cell with that price. The steps below are the pages to follow, in order.

::diagram{id=journey-rehearse-a-crisis}

## Who takes part

- **The exercise planner** builds and runs the exercise in OpenAEV: the timeline, the crisis cell, the documents, the newspaper, the article, the challenge, then the debrief. Every step but two is theirs.
- **The risk manager** checks in OpenCRQ what the breach costs, at the step [Know what the breach costs](/opencrq/use/decide/dashboards/), so the figure the cell reads is one they stand behind.
- **The AI agent builder** creates the XTM One agent that briefs the cell and shares it with the cell, at the step [Brief the cell with an agent](/xtm-one/build/agents/).
- **The crisis cell**, led by the CISO (chief information security officer), plays: it receives the emails, answers the challenge, reads the article, asks the agent, and fills in the questionnaire.

## Why it spans several products

A crisis exercise trains people, not machines, so OpenAEV plays it to them: each inject reaches the players of a team by email at its time, and each expectation scores how they react. See [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/).

What a crisis cell decides depends on what the breach costs: how many guests to notify, which message to publish, what to escalate to the executive committee. OpenCRQ gives that cost from its risk scenarios, priced in money. See [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

No link carries that figure from OpenCRQ to OpenAEV. XTM One carries it to the people instead: an agent bound to the OpenCRQ integration reads the exposure as the person who asks, searches the crisis playbook in a knowledge base, and drafts the briefing. See [What agents are made of](/xtm-one/concepts/agents/). Without it, the cell plays the timeline without the numbers it would have on the real night.

## What you need

- **OpenAEV**, any edition. The platform sends email, and the members of the crisis cell have an email address. **Ask AI**, which drafts the article, requires the Enterprise Edition and XTM One.
- **OpenCRQ**, with the risk scenario the exercise is about already priced. See [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- **XTM One**, with the Enterprise Edition and AI switched on: agents and presentations are part of it. OpenCRQ and OpenCTI are registered with it, so their integrations exist. See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one).
- The same email address for each member of the crisis cell in XTM One and in OpenCRQ: the agent reads OpenCRQ as the person who asks, and an account with no match gets no data.
- The capabilities, by their label in each product's role editor:
  - In OpenAEV, the exercise planner has **Manage assessment** and **Launch assessment**, **Manage teams & persons**, **Manage documents**, **Manage channels**, **Manage challenges**, and **Manage lessons learned** to create the questionnaire's categories.
  - In OpenCRQ, the risk manager needs no capability to read the dashboards.
  - In XTM One, any user creates an agent and shares it; only an administrator makes it a **Company-managed agent**.

## How it unfolds

Example CERT plays the Velvet Harbor intrusion of 9 January 2026 again to the crisis cell of Grand Meridian Hotels, in the scenario `Velvet Harbor crisis night`.

**Plan the timeline.** The exercise planner creates a time-based scenario, category Global Crisis, with lessons learned turned on. Each launch copies it into a simulation, so the same exercise can be played again to another cell, and each run keeps its own answers. See [Plan the exercise](/openaev/concepts/scenarios-simulations/).

**Set the stage.** The planner adds the team **Crisis cell**, with Hélène Marchetti, Sophie Laurent and Marc Dupuis ([Gather the crisis cell](/openaev/use/targets/people/)). They upload the crisis communication playbook, the Meridian EDR alert of the front-desk workstation and a fake screenshot of the forum listing ([Share the alert and the playbook](/openaev/use/components/documents/)). They create the newspaper **Hotel Business Daily** ([Create the press outlet](/openaev/use/components/channels/)), and write its article `Guest records of Grand Meridian Hotels offered on a criminal forum` ([Publish the article](/openaev/use/components/media-pressure/)).

**Write the night.** The timeline starts at 22:00. At 40 minutes, the CISO's email calls the cell to the bridge, with the playbook attached. At 50 minutes, the challenge `Find the account that was used` gives the cell the EDR alert and asks which account ran the loader: `night.audit` is the answer ([Ask the cell to investigate](/openaev/use/components/challenges/)). At 1 hour 30, Hotel Business Daily publishes the forum listing.

**Price the breach.** The risk manager opens OpenCRQ on the risk scope Grand Meridian France. The risk scenario Data Breach on Guest Records reads $33K of loss a year: the yearly average, low because such a breach is rare. One incident costs $59.0M expected, notification of 2.4 million guests included ([Know what the breach costs](/opencrq/use/decide/dashboards/)). The second figure is the one a crisis cell needs on the night: see [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).

**Prepare the briefing.** Sophie Laurent, the AI engineer, builds the agent **Crisis briefer** in XTM One. It is bound to the OpenCTI and OpenCRQ integrations, searches the knowledge base **Grand Meridian security policies**, which holds the playbook, and is shared with the group **Crisis cell** in **Read only** ([Brief the cell with an agent](/xtm-one/build/agents/)). It drafts the facts known so far, the messages to guests, staff and press from the playbook, and what the exposure costs according to OpenCRQ. It can turn them into a deck: see [Turn an agent's work into a presentation](/xtm-one/use/presentations/).

**Play it.** The planner launches the simulation. The **Mails** menu of its **Execution** tab shows the cell's replies. Marc Dupuis isolates the booking database server; Sophie Laurent answers that Crisis briefer is drafting the first situation report. Whether the cell was on the bridge within 30 minutes is an expectation the control team scores by hand. The simulation stays **On-going** until every expectation is answered or expired ([Run the exercise](/openaev/use/simulations/)).

**Debrief.** The morning after, the planner applies the template **Crisis exercise debrief** and sends its questionnaire to the cell. **Communication** averages 40: nobody owned the answer to the journalist. That gap opens the action plan ([Collect the lessons learned](/openaev/use/components/lessons/)).

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Test whether your staff fall for the lure](/workflows/test-your-people/): send the front desks a lure like the one that opened the intrusion.
- [Check your defenses against the threats you track](/workflows/threat-informed-validation/): replay the same intrusion on the endpoints, and read what the EDR stopped.
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/): how the reactions of the players are scored.
- [Add expectation results by hand](/openaev/use/expectations/validation/): the reactions of people your control team scores.
