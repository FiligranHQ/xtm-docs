---
title: Keep indicators from decaying
description: Keep some OpenCTI indicators out of decay, such as detection rules or feeds that manage their own lifecycle, with decay exclusion rules.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [soc-manager, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/decay-exclusion-rules/
---

A decay exclusion rule keeps matching indicators out of decay: their score never falls and they are never revoked automatically. Use one for indicators that must stay active, such as YARA or Sigma detection rules, and for sources that already manage the lifecycle of their indicators. Read [Age indicators with decay rules](/opencti/configure/decay/) first for how decay works.

## Before you start

- Your role has the **Manage customization** capability.
- You know how to recognize the indicators to exclude: their author, creator, marking, label, main observable type, indicator type or pattern type.

:::warning
Once an indicator falls under an exclusion rule, it stays there: deleting the rule or changing its filters does not bring the indicator back into decay. Test your filters before you activate a rule.
:::

## Create an exclusion rule

:::steps

1. Open :menu[Settings > Customization > Decay rules] and select the **Decay exclusion rules** tab.
2. Click **Create Decay Exclusion Rule**. The **Create a decay exclusion rule** panel opens.
3. Give the rule a **Name** and a **Description** that says why these indicators are excluded.
4. Add at least one filter. The keys are **Creator**, **Author**, **Marking**, **Label**, **Main observable type**, **Indicator type** and **Pattern type**.

   :::only{opencti=">=7.261117"}
   The keys also offer the [custom fields](/opencti/configure/taxonomies/#custom-fields) added to the **Indicator** type, by their label.
   :::

5. Leave **Active** off for now and click **Create**. A rule is created inactive unless you turn on **Active**.
6. Check the filters once more, then open the menu of the rule in the list and choose **Enable**.

:::

::screenshot{id=opencti-decay-exclusion-create alt="The Create a decay exclusion rule panel, with its warning about rules without filters, the filter selector and the Active switch turned off."}

:::warning
A rule without filters matches every indicator, so no indicator would ever expire again. The creation panel warns you about it.
:::

:::example[Keep Example CERT detection rules active]
Example CERT writes YARA and Sigma rules for the files and logs of Velvet Harbor intrusions. A rule stays valid as long as the malware does not change, so its score must not fall like the score of an IP address.

- **Name**: `Example CERT detection rules`
- **Description**: `YARA and Sigma rules written by our analysts: reviewed by hand, never expired`
- Filters: **Pattern type** is `yara` or `sigma`, and **Author** is `Example CERT`
- **Active**: off at creation, then **Enable** from the row menu once the filters are checked

YARA and Sigma rules from other authors keep decaying under the built-in default rule, and STIX indicators of Example CERT, such as the IP addresses of Velvet Harbor, still expire. On a Velvet Harbor YARA rule created afterwards, the lifecycle button names **Example CERT detection rules**.
:::

## How exclusion rules apply

- **No order.** Exclusion rules are not ranked like decay rules. An indicator that matches any active exclusion rule is excluded, so several rules add up as alternatives.
- **Before decay rules.** When an indicator matches both an exclusion rule and a decay rule, the exclusion rule wins.
- **Revoked or not.** An indicator is checked against exclusion rules even when its source creates it revoked.

When OpenCTI evaluates the rules depends on what happens to the indicator:

| Event | What happens |
| --- | --- |
| Creation | The indicator falls under an exclusion rule if it matches one; otherwise under a decay rule |
| Upsert: a source sends the same indicator again | An indicator that now matches an exclusion rule moves under it, for example after a new label or when the creator in your filter sends it. An indicator already excluded stays excluded |
| Update by a user or a playbook | Nothing changes: an indicator under a decay rule stays under it, even if it now matches an exclusion rule |

For example, with a rule that filters on the creator of a trusted feed, an indicator first created by another source decays normally, until the trusted feed sends it too. From then on it is excluded.

## What your detection tools receive

An excluded indicator keeps the score its source or an analyst sets: no reaction point lowers it, and decay never revokes it. Decay computes no **Valid until** for it either. A **Valid until** that its source sets still applies, and the indicator is revoked when that date passes. So an exclusion rule keeps indicators in every feed and stream that selects them until someone revokes them or that date passes. Filter those feeds on the author or the label if some consumers must not get them.

## See which rule applies to an indicator

On an indicator, the lifecycle button next to its score differs when the indicator is excluded. Click it to see the name of the exclusion rule; if you can manage customization, the name links to the rule.

## Change, disable or delete a rule

Open the menu of a rule in the list: **Update** opens **Update a decay exclusion rule**, where you change its name, description and filters, **Disable** (or **Enable**) switches it, **Delete** removes it. None of these change indicators that are already excluded; they only affect indicators evaluated from then on.

## What's next

- [Age indicators with decay rules](/opencti/configure/decay/) for the indicators that should decay.
- [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/), what analysts see on an indicator.
