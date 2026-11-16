---
title: Age indicators with decay rules
description: Create OpenCTI decay rules that lower indicator scores over time and revoke stale indicators, and choose which indicators each rule applies to.
sidebar_position: 5
doc_type: how-to
products:
  - opencti
personas: [soc-manager, admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/decay-rules/
---

A decay rule lowers the score of indicators as they age and revokes them when the score gets too low, so that your SIEM and EDR stop alerting on stale indicators. OpenCTI ships with built-in rules for the common observable types; this page shows how to add your own, for example a faster decay for a noisy feed, and what your detection tools receive at each step. What the score means for analysts is in [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/).

## Before you start

- Your role has the **Manage customization** capability.
- The indicator decay manager is running. It is on by default (`indicator_decay_manager:enabled` in the [configuration reference](/reference/opencti/configuration/)); when it is off, the page says so and no score changes.

## How a rule is chosen

OpenCTI picks the rule of an indicator once, when the indicator is created:

::diagram{id=opencti-decay-rule-selection}

1. If an active [decay exclusion rule](/opencti/configure/decay/exclusions/) matches, the indicator never decays.
2. Otherwise, among the active decay rules that match the indicator, the one with the highest **Order** applies.
3. An indicator created already revoked gets no decay rule.

Then the rule sticks. Changing the filters of a rule, or editing the indicator so that it matches another rule, does not move indicators that already have a rule. Only new indicators follow the new settings, with one exception: when a source sends an existing indicator again with a new score, and its confidence allows the update, the decay restarts from that score under the rule that matches at that moment.

## The built-in rules

Four rules come with the platform: one for IP address and URL indicators, one for domain names, one for files and artifacts, all with order 1, and a default rule with order 0 for every other indicator. They are active and cannot be changed or deleted. Their lifetime, decay factor, reaction points and revoke score are in the [decay rule reference](/reference/opencti/decay-rules/).

To replace a built-in rule, create a rule that matches the same indicators with a higher order: 2 or more. The list in :menu[Settings > Customization > Decay rules] shows the **Order** of every rule, its **Creation date**, whether it is **Active**, and how many **Impacted indicators** it holds.

::screenshot{id=opencti-decay-rules-list alt="The list of decay rules: a custom rule with order 5 above the four built-in rules with order 1 and 0, each with its number of impacted indicators."}

## What your detection tools receive

Decay changes the indicator itself, so every consumer sees it: [live streams](/opencti/use/share/feeds/#create-a-live-stream) send each change as an update, and [CSV feeds and TAXII collections](/opencti/use/share/feeds/) apply their filters to the new values.

- **The score changes only at reaction points.** Between two points, the indicator keeps its last score, even though the life curve falls continuously. A rule without reaction points leaves the score untouched until the indicator is revoked.
- **Valid until is set at creation** to the date the score will reach the revoke score.
- **At that date, the indicator is revoked**: **Revoked** becomes true, **Detection** becomes false and the score drops to the revoke score.
- **Changes land within minutes.** The indicator decay manager runs every minute and updates up to 10,000 indicators per pass (`indicator_decay_manager:interval` and `indicator_decay_manager:batch_size`). The expiration manager revokes indicators past their **Valid until** every 5 minutes (`expiration_scheduler:interval`). When more indicators are due at once, the backlog takes several passes.

So place a reaction point on every score threshold your integrations filter on. If the EDR feed keeps indicators with a score greater than 60, a reaction point at 60 takes an indicator out of the feed when the curve reaches 60; without it, the indicator stays in the feed until the next point or its revocation.

## Create a decay rule

:::steps

1. Open :menu[Settings > Customization > Decay rules] and click **Create Decay Rule**. The **Create a decay rule** panel opens.
2. Give the rule a **Name** and a **Description** that says which indicators it is for.
3. Choose the indicators it applies to.

   :::only{opencti=">=7.260811"}
   Add filters on any of these keys: **Author**, **Creator**, **Marking**, **Label**, **Main observable type**, **Indicator type** and **Pattern type**. A rule without filters applies to every indicator.
   :::

   :::only{opencti="<7.260811"}
   Pick the main observable types in **Apply on indicator observable types (none = ALL)**. A rule without types applies to every indicator.
   :::

4. Set the decay curve:
   - **Lifetime (in days)**: how long the score takes to fall from its initial value to 0. It starts at 365;
   - **Decay factor**: the shape of the curve. It starts at 1; a higher factor makes the score fall faster at the start, a lower one keeps it high longer. The built-in rules use 0.3 to 0.7.
5. Add **Reaction points**: click **Add a reaction point** for each score at which OpenCTI updates the indicator, and **Remove this reaction point** to drop one. With points at 60 and 40, an indicator created at 80 is updated to 60, then to 40, when the curve reaches them. Keep every point above the revoke score.
6. Set the **Revoke score**: the indicator is revoked when its score reaches this value. It starts at 0, which revokes the indicator only at the end of its lifetime.
7. Set the **Order**. It starts at 1, the order of the built-in rules: give the rule a higher order than any other rule that matches the same indicators.
8. Turn on **Active**, or leave it off to activate the rule later, and click **Create**. A new rule is inactive unless you turn it on.

:::

::screenshot{id=opencti-decay-rule-create alt="The Create a decay rule panel with the filters, Lifetime (in days), Decay factor, Reaction points, Revoke score, Order and Active fields."}

::::example[Expire scanner IP addresses within days]
The Example Scanner Feed sends thousands of IPv4 addresses of mass scanners a day, each with a score of 50. Under the built-in IP and URL rule they stay active for about 20 days, long after the scanners have moved on, and the SOC gets alerts on addresses that no longer scan anyone.

- **Name**: `Noisy feed IP addresses`
- **Description**: `Faster decay for IP indicators from the Example Scanner Feed`

:::only{opencti=">=7.260811"}
- Filters: **Main observable type** is `IPv4 address`, and **Author** is `Example Scanner Feed`, so indicators from other sources keep the built-in rule.
:::

:::only{opencti="<7.260811"}
- **Apply on indicator observable types (none = ALL)**: `IPv4 address`. This version cannot filter on the author, so the rule applies to IPv4 indicators of every source.
:::

- **Lifetime (in days)**: `30`
- **Decay factor**: `0.8`, a steep fall at the start
- **Reaction points**: `70` and `40`
- **Revoke score**: `20`
- **Order**: `5`, above the built-in rule (order 1)
- **Active**: on

An indicator the feed creates at score 50 then goes through these steps; the **Life curve** on the page of the rule draws the same curve:

```text
Day 0      score 50   created by Example Scanner Feed, rule "Noisy feed IP addresses"
Hour 15    score 40   reaction point: the indicator is updated
Day 9      score 20   revoke score: the indicator is revoked, detection tools drop it
```

An indicator created at 80 by the same feed reaches 70 within hours, 40 after about six days and is revoked after about 15 days.
::::

When two rules that match the same indicator have the same order, OpenCTI picks one by an internal identifier drawn at random when each rule was created: the choice stays the same on a given platform, but it is neither the oldest rule nor one you can predict. Give each custom rule its own order.

## Check a rule

Click a rule in the list to open its page. **Configuration** recalls its settings, **Impact** counts the indicators **currently impacted by this rule**, and **Life curve** draws how the score falls over time, with the reaction points and the revoke score.

:::only{opencti=">=7.260811"}
In **Configuration**, **Decay indicator filter** shows the filters of the rule. When it is empty, the rule matches every indicator.
:::

:::only{opencti="<7.260811"}
In **Configuration**, **Indicator observable types** lists the main observable types of the rule. When it is empty, the rule matches every indicator.
:::

::screenshot{id=opencti-decay-rule-detail alt="The page of a custom decay rule with its configuration on the left, the number of impacted indicators, and the life curve falling from 100 to the revoke score."}

On an indicator, the lifecycle button next to its score shows the rule applied and the dates of the next score changes. See [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/).

## Change, deactivate or delete a rule

On the page of a custom rule, **Update** opens **Update a decay rule**, where you change any setting, and the menu next to it offers **Deactivate** (or **Activate**) and **Delete**. Built-in rules have neither. A change applies to the indicators created afterwards; indicators already under the rule keep the settings they got. An inactive rule is skipped when new indicators are created; indicators already under it keep it.

To keep some indicators out of decay altogether, open the **Decay exclusion rules** tab of the same screen: see [Keep indicators from decaying](/opencti/configure/decay/exclusions/).

## When two sources disagree on a score

Two sources of equal trust can keep resetting the score of the same indicator. To stop that loop, a source cannot set a score it already set in the past. For example: source 1 creates an indicator at 90, source 2 raises it to 95, then source 1 sends 90 again: that last update is ignored. You can also lower the confidence level of the user of one source, so that the other one wins. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

## What's next

- [Keep indicators from decaying](/opencti/configure/decay/exclusions/) for indicators that must never expire.
- [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/), what analysts see.
- [How OpenCTI works](/opencti/concepts/architecture/) for the managers that run in the background.
