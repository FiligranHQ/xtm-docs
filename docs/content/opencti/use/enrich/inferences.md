---
title: How OpenCTI infers new relationships
description: How the OpenCTI rules engine derives new relationships from existing ones, how to recognize inferred knowledge, and what the inference rules do.
sidebar_position: 2
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, soc-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/inferences/
---

Some relationships in OpenCTI were written by no one: the rules engine deduced them from the knowledge you entered. If an intrusion set targets France and France is located in Europe, the engine states that the intrusion set targets Europe, so a search for threats against Europe finds it. This page explains how these inferences work, how to tell them from what analysts entered, and why you cannot edit them. Administrators turn each rule on or off in :menu[Settings > Customization > Rules engine]; analysts read the result.

## What it is good for

| Goal | Example |
| --- | --- |
| Complete victimology | Targeting of a country rolls up to its region, targeting of a subsidiary to its parent organization |
| Follow attributions | What an intrusion set uses or targets is also true of the threat actor it is attributed to |
| Turn sightings into alerts | A sighting of a valid indicator in your systems raises an incident |
| Keep reports self-contained | A report that contains an indicator also gets the observable it is based on |

:::only{opencti=">=7.261030"}
Vulnerability propagation rules also give a vulnerability to the systems that use a vulnerable software and to the infrastructures that consist of it.
:::

## How it works

::diagram{id=opencti-inferences-rule}

A rule is a statement of the form IF these relationships exist, THEN create this one. When an administrator activates a rule, the engine first applies it to all existing data, then to every change as it happens.

Three behaviors surprise people:

1. **Inferred knowledge follows its facts.** When a relationship a rule relied on is deleted, the inferred relationship is deleted too. You cannot edit or delete an inferred relationship yourself; change the facts instead.
2. **Inferred relationships combine their sources.** They carry the markings of all the facts they come from, and the average of their confidence levels.
3. **One relationship can have several explanations.** When two rules, or the same rule from two sets of facts, lead to it, it stays as long as one explanation holds.

## Recognize inferred knowledge

- **In graphs**, an inferred relationship is a dashed line in the warning color, and an inferred entity carries a wand badge. See [Work with graphs](/opencti/use/analyze/graphs/).
- **In lists**, a wand icon at the end of the line marks an inferred relationship.
- **On the relationship page**, the description reads **Inferred knowledge**, there is no edit button, and the **Inference explanation** section shows the rule and draws the facts that produced it.

::screenshot{id=opencti-inferences-explanation alt="The page of an inferred targets relationship, with Inferred knowledge as its description and the Inference explanation section drawing the two relationships the rule used."}

## The inference rules

Each rule is off until an administrator activates it. The rules fall into categories that match the goals above: alerting, correlation, parent-child propagation, victimology and report propagation.

:::only{opencti=">=7.261030"}
A sixth category, vulnerability propagation, spreads a vulnerability from a software to the systems that use it and the infrastructures that consist of it.
:::

Three examples:

| Rule | IF | THEN |
| --- | --- | --- |
| Location propagation | Location A is located at B, and B at C | A is located at C |
| Usage propagation via attribution | A uses B, and A is attributed to C | C uses B |
| Observables propagation in reports | A report contains an indicator based on an observable | The report also contains the observable and the relationship |

Report propagation rules apply to reports only, not to other containers.

:::example[Find Velvet Harbor among the threats against Europe]
Your reports say that Velvet Harbor targets France and Spain, and nobody wrote that it targets Europe. An administrator has activated **Targeting propagation via location**. Open Velvet Harbor in :menu[Threats > Intrusion sets], then its **Knowledge** tab: a **targets** relationship to Europe shows with the wand icon. Open it: its description reads **Inferred knowledge**, and **Inference explanation** draws the facts behind it, Velvet Harbor targets France and Spain, both located in Europe. A colleague who lists the threats targeting Europe now finds Velvet Harbor. If an analyst deletes the targeting of France as wrong, the relationship to Europe stays, explained by Spain. It disappears only when no explanation is left, and inferences chain: Velvet Harbor also targets the booking database server of Grand Meridian Hotels, located in France, so the platform still infers that it targets France, and from there Europe.
:::

The [inference rule reference](/reference/opencti/inference-rules/) lists every rule of your version with what it infers.

## Rules and the alternatives

| You want to | Use |
| --- | --- |
| Derive the same kind of relationship everywhere, always | An inference rule |
| Apply a rule to some data only, or once | A playbook with **Apply predefined rule**, or a [background task](/opencti/use/automate/background-tasks/) |
| Add knowledge from an external source | An [enrichment connector](/opencti/use/enrich/connectors/) |

## What's next

- [Turn inference rules on or off](/opencti/configure/rules/), to activate rules and follow their progress.
- The [inference rule reference](/reference/opencti/inference-rules/), every rule and its pattern.
- [Work with graphs](/opencti/use/analyze/graphs/), where inferred knowledge is drawn.
- [Automate with playbooks](/opencti/use/automate/playbooks/), to apply a rule to selected data.
