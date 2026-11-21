---
title: Turn inference rules on or off
description: Turn OpenCTI inference rules on or off, know what each rule creates, and weigh the effect of a rule on your data before you enable it.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [admin, cti-lead]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/reasoning/
---

The rules engine derives new knowledge from what is already in OpenCTI. Each inference rule watches for a pattern, for example "A is part of B and B is part of C", and creates the missing link, here "A is part of C". This page shows how to enable a rule, what the rules create, and what to check first. CTI team leads find what each family of rules changes for their analysts in [Choose the rules with the CTI team](#choose-the-rules-with-the-cti-team). How analysts read inferred knowledge is in [Inferences](/opencti/use/enrich/inferences/).

## Before you start

- Your role has the **Manage customization** capability.
- To switch a rule, your role also needs **Allow modification of sensitive configuration**: rules are a protected setting by default, marked **Danger Zone**, unless the platform turns that protection off. See [Sensitive configuration](/opencti/administer/sensitive-configuration/).
- The rule engine manager is running. It is on by default (`rule_engine:enabled` in the [configuration reference](/reference/opencti/configuration/)); when it is off, the page says so and shows no rules.

## Enable or disable a rule

:::steps

1. Open :menu[Settings > Customization > Rules engine]. The top of the page shows the **Total inferred entities** and **Total inferred relations** with their change over 24 hours, the **Rules engine status** and the **Last event processed**.
2. Find the rule. Rules are grouped by category, each with its **Description** and an **IF** / **THEN** diagram of what it creates.
3. Turn on the **Status** switch of the rule and confirm with **Enable**.

:::

::screenshot{id=opencti-rules-list alt="The Rules engine page with the Raise incident based on sighting rule: its description, its Status switch, and the IF and THEN diagram of what it creates."}

A progress bar under the switch shows **Applying this rule on the existing data**, then **This rule has been applied on the existing data**.

Turning a rule off works the same way, with **Disable**. The progress bar shows **Cleaning up this rule on the existing data**.

:::example[Show what Harbor Syndicate targets]
Example CERT attributes the intrusion set Velvet Harbor to the threat actor group Harbor Syndicate. Velvet Harbor targets the hospitality sector, France and Spain, but the page of Harbor Syndicate shows no target: analysts who start from the group miss its victims.

In :menu[Settings > Customization > Rules engine], under **Victimology**, turn on **Targeting propagation via attribution** and confirm with **Enable**. Once the progress bar reads **This rule has been applied on the existing data**, the rule has drawn:

```text
Known:     Velvet Harbor  -- attributed to -->  Harbor Syndicate
Known:     Velvet Harbor  -- targets -->        Hospitality, France, Spain
Inferred:  Harbor Syndicate  -- targets -->     Hospitality, France, Spain
```

The **Victimology** of Harbor Syndicate now lists the sector and the two countries, each relationship marked as inferred and explained by the two facts above. **Total inferred relations** grows by the same number. If an analyst deletes the attribution, the rule removes the three inferred relationships.
:::

## What happens when you switch a rule

- **On**: a background task scans the whole knowledge base and creates what the rule infers from existing data. From then on, the rule runs each time an object or relationship that matches its conditions is created or changed.
- **Off**: a background task deletes everything the rule created. Knowledge that users or connectors created is never touched.

Inferred objects and relationships are marked as inferred, and each one lists the rule and the facts it comes from. When a fact a rule relied on disappears, the rule removes what it inferred from it.

:::warning
A rule can create a very large number of objects on a big knowledge base, and the initial scan loads the platform. Enable one rule at a time, outside busy hours, and watch the inferred counters before enabling the next. Disabling a rule undoes its work, but the clean-up also takes time.
:::

## The inference rules

The page groups the rules by category, described with what they change for analysts in [Choose the rules with the CTI team](#choose-the-rules-with-the-cti-team). A few examples of what they create:

| Rule | If | Then |
| --- | --- | --- |
| Belonging propagation | A is part of B, and B is part of C | A is part of C |
| Targeting propagation via attribution | A targets B, and A is attributed to C | C targets B |
| Raise incident based on sighting | A non-revoked Indicator is sighted in an entity | An Incident linked to the Indicator and to the entity |

The [inference rule reference](/reference/opencti/inference-rules/) lists every rule of your version with its **IF** / **THEN** pattern and its identifier. The **Relation propagation testing rule**, which propagates "related to" across the whole graph, only runs in development mode.

## Choose the rules with the CTI team

A rule changes what every analyst sees: the page of a threat actor gains targets, a report gains objects nobody added by hand. Decide which rules to turn on with the CTI team lead, from the way the team models its knowledge.

| Category | What analysts get | Turn it on when |
| --- | --- | --- |
| **Alerting** | Incidents raised from sightings of valid indicators, and sightings carried between an indicator and its observables | The SOC records sightings in OpenCTI and wants each one followed up |
| **Correlation** | "Related to" relationships between entities that share an observable or an infrastructure | Analysts pivot on shared infrastructure. A common observable, such as a public DNS resolver, can relate many unrelated entities |
| **Parent-child propagation** | Attributions, belongings, locations and usages carried up a chain, such as the techniques of an intrusion set shown on the threat actor it is attributed to | The team attributes intrusion sets to threat actors, or links sub-techniques, and wants the parent's page complete |
| **Report propagation** | Objects implied by a report's content added to it, such as the parent location of a city or the indicators of an observable | Readers of a report should find the full context without opening each object |
| **Victimology** | Targets carried to the attributed actor, to the parent organization or to the parent location | The team follows what an actor or a sector is targeted by at every level |

:::only{opencti=">=7.261030"}
**Vulnerability propagation** rules give systems and infrastructures the vulnerabilities of the software they use: turn them on when the platform tracks your own assets.
:::

Tell analysts which rules are on. Inferred knowledge follows its sources: when an analyst corrects an attribution, what the rule inferred from it goes away too.

## Suggest a new rule

You cannot write your own rules, because of the effect a rule can have on the data. To suggest one, open an issue on the [OpenCTI GitHub repository](https://github.com/OpenCTI-Platform/opencti/issues) with the pattern and what it should create.

## What's next

- [Inferences](/opencti/use/enrich/inferences/), how analysts see and use inferred knowledge.
- The [inference rule reference](/reference/opencti/inference-rules/), every rule and what it infers.
- [Automate with playbooks](/opencti/use/automate/playbooks/), to apply one rule to selected data only.
- [Sensitive configuration](/opencti/administer/sensitive-configuration/), who may switch rules.
