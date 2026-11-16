---
title: How indicators age and expire
description: What the score, the validity dates, the decay and the revocation of an OpenCTI indicator mean for detection, and what happens when you or a source change them.
sidebar_position: 5
doc_type: explanation
products:
  - opencti
personas: [soc-analyst, soc-manager, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/indicators-lifecycle/
---

An indicator in OpenCTI does not stay in your detection tools forever. Its score falls as it ages, following a decay rule picked for its type: an IP address indicator created at a score of 80 drops to 50 after about 9 days and is revoked after about 29. A revoked indicator leaves the feeds and streams that select valid indicators. This page explains how to read where an indicator stands, why it left your SIEM, and what each change you make does to it.

You find indicators in :menu[Observations > Indicators]. Changing the score or the dates of an indicator needs **Create / Update knowledge**, and deleting it **Delete knowledge**; changing the decay rules needs **Manage customization** and is described in [Age indicators with decay rules](/opencti/configure/decay/).

## Read an indicator

The **Details** card of an indicator answers "is it still in my detection tools, and for how long?":

| Field | What it tells you |
| --- | --- |
| **Score** | How much to trust the indicator today, from 0 to 100. It falls by steps as the indicator ages. Feeds and streams often keep only indicators above a score, such as 60 |
| **Lifecycle** | The button next to the score, when a decay rule or a decay exclusion rule applies. It opens **Lifecycle details**, the curve of the score from its **Initial score** to its **Revoke score**, with the **Current stable score** and the history of changes |
| **Valid from** | When the indicator started to be valid |
| **Valid until** | When the indicator is expected to be revoked. Past this date, it is revoked |
| **Detection** | Whether the indicator is meant for detection tools. Sources and analysts set it, and revoking the indicator sets it to **No** |
| **Main observable type** | The kind of value the indicator detects, such as an IPv4 address or a URL. It decides which built-in decay rule the indicator follows |

In the list, filter on **Revoked**, **Score**, **Valid until** or **Detection** to see what your feeds send: for example, **Revoked** equals `false` and **Score** greater than or equal to `60`.

::screenshot{id=opencti-indicators-lifecycle-dialog alt="The Lifecycle details dialog of a URL indicator: a curve from the initial score of 100 down to the revoke score of 20, the reaction points marked on it, and the history of score changes."}

## How long an indicator stays active

OpenCTI picks a decay rule when the indicator is created and keeps it for the indicator's whole life. The built-in rules follow how fast each kind of value goes stale. Days count from **Valid from**:

| Rule | Main observable of the indicator | Score 100 at creation | Score 80 at creation | Score 50 at creation |
| --- | --- | --- | --- | --- |
| Built-in IP and URL | IPv4 or IPv6 address, URL | 80 at day 3, 50 at day 15, revoked at day 32 | 50 at day 9, revoked at day 29 | Revoked at day 20 |
| Built-in domain name | Domain name | 80 at day 10, 50 at day 70, revoked at day 188 | 50 at day 38, revoked at day 164 | Revoked at day 103 |
| Built-in files and artifact | File, artifact | 80 at day 108, revoked at day 376 | Revoked at day 355 | Revoked at day 290 |
| Built-in default | Anything else | 80 at day 87, 50 at day 227, revoked at day 372 | 50 at day 168, revoked at day 348 | Revoked at day 275 |

The score changes only at the reaction points of the rule, 80 and 50 for most of them, and the indicator is revoked when its score reaches the revoke score, 20. An indicator without a score from its source starts at 50. Your administrator can change the lifetimes or add rules for a source: see [Age indicators with decay rules](/opencti/configure/decay/) and the [decay rule reference](/reference/opencti/decay-rules/).

## How it works

::diagram{id=opencti-indicators-lifecycle}

**At creation**, OpenCTI:

1. Picks the active decay rule that matches the indicator and stores it with the indicator. Changing or adding decay rules later only affects new indicators.
2. Starts the score at the value the source gives, or 50.
3. Sets **Valid from** to the date the source gives, or the creation date of the indicator, or now.
4. Sets **Valid until** to the moment the score will reach the revoke score, replacing any date the source gave. An indicator created already revoked, because its validity is over, keeps its dates and gets no decay.

**Over time**, the decay manager checks every minute for indicators that reach a reaction point and lowers their score. When the score reaches the revoke score, the indicator is revoked: **Detection** becomes **No**, and every feed and stream that selects valid indicators drops it. A live stream sends the change as an update, so a connected SIEM can remove the indicator; a CSV feed or a TAXII collection leaves it out of the next download.

The **Sightings** tab lists where your tools saw the indicator, with **First seen**, **Last seen** and **Count**. Click **Create a sighting** to record a hit by hand, and set **False positive** on a sighting your investigation shows benign. A sighting, true or false positive, does not change the score: lower it or revoke the indicator yourself. Your tools can keep sighting a revoked indicator, but the rule **Raise incident based on sighting** only raises incidents for valid indicators.

## When the indicator changes

When you or a source edit an indicator, the lifecycle reacts to its score and its validity:

| Change | Result |
| --- | --- |
| A new score | The decay restarts from the new score with the same rule, and **Valid until** moves forward |
| The score the current cycle started from | Nothing changes. Choose another score to restart the decay |
| **Valid until** set in the past | The indicator is revoked now: its score drops to the revoke score and **Detection** becomes **No** |
| **Revoked** set to true by a source or through the API, with no score or date in the same change | The same as **Valid until** in the past. The form has no **Revoked** field: connectors and scripts set it |
| **Revoked** set to false by a source or through the API, with no score or date in the same change | The decay restarts from the score its cycle started from, as for **Valid until** in the future. A score sent in the same change wins over it |
| **Valid until** set in the future on a revoked indicator | The indicator is valid again and its decay restarts from the score its cycle started from. OpenCTI recomputes **Valid until** from the rule, so the date you typed is replaced |
| **Valid until** set later on a valid indicator | The date changes, but the decay does not stop: the indicator is still revoked when its score reaches the revoke score |

When a source sends an indicator that already exists (an upsert, see [Deduplication](/opencti/concepts/deduplication/)), a new score restarts the decay in the same way. A score that this source already set on the indicator leaves the score and the dates as they are, so a feed that resends the same indicator every day does not keep it alive.

:::only{opencti=">=7.260811"}
The same goes for a source that resends the indicator with its current score, such as a playbook that ingests it again.
:::

:::only{opencti=">=7.261030"}
A source whose confidence is lower than the indicator's cannot change its score or its dates at all. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).
:::

**Two sources that disagree** could keep overwriting each other's score and restart the decay forever. To prevent this, OpenCTI ignores a score when the same source already set that exact score on the indicator. For example, source A sets 40, source B sets 35, then A sets 40 again: the last update is ignored. Had A sent 45, it would have been accepted.

An indicator with no decay rule, because decay was off or the indicator was created revoked, follows fixed values: revoking it sets its score to 0, and making it valid again sets the score to 50 and keeps it valid for 90 days.

## What you can do on an indicator

Open the indicator and click **Update**:

- **Take it out of detection now**, after a false positive: set **Valid until** to now or a moment in the past. The indicator is revoked when you save, and the streams send the change.
- **Keep it active longer**, because the threat is still live: raise its **Score**. The decay restarts from the new score. Moving **Valid until** later is not enough.
- **Bring back a revoked indicator**: set a new **Score**, or a **Valid until** in the future. Then check **Detection**: making the indicator valid again does not set it back to **Yes**, so set it yourself if your feeds select on it.

Revoke rather than delete. A revoked indicator keeps its sightings and its history, and a feed that resends it with the same score leaves it revoked. A deleted indicator loses its relationships, and the next time a source sends it, it comes back as a new indicator with a fresh decay. Deleting, from the actions menu of the indicator, needs **Delete knowledge**; see [Delete and restore](/opencti/use/delete-restore/).

To change many indicators at once, use a [background task](/opencti/use/automate/background-tasks/) on a filtered list.

:::example[Find out why a phishing URL left the SIEM]
On a Monday, the SOC notices that `https://booking-velvetharbor.example/login` no longer alerts on the web proxy, while Velvet Harbor is still active. Open :menu[Observations > Indicators], search for the URL, and open the indicator.

Its **Score** reads 50, not the 100 Example CERT gave it. Click **Lifecycle**: the indicator follows **Built-in IP and URL**, it dropped from 100 to 80 after three days and to 50 after fifteen. The proxy feed keeps indicators scored 60 and above, so the URL left it at day 15. It would have been revoked at day 32.

The CTI team confirms that the domain still serves the fake booking page. Click **Update** and set the **Score** to `90`. The decay restarts from 90 with the same rule, **Valid until** moves forward, and the URL is back in the proxy feed at the next download.
:::

## Decay exclusion and the alternatives

| You want to | Use |
| --- | --- |
| Make a kind of indicator expire faster or slower | A [decay rule](/opencti/configure/decay/) |
| Keep some indicators valid with no decay, such as your own blocklist or your YARA rules | A [decay exclusion rule](/opencti/configure/decay/exclusions/). Like a decay rule, it is checked at creation only |
| Stop OpenCTI from creating indicators for known benign values | An [exclusion list](/opencti/configure/exclusion-lists/) |
| Change the score of many indicators at once | A [background task](/opencti/use/automate/background-tasks/) or a [playbook](/opencti/use/automate/playbooks/) |

An indicator under a decay exclusion rule shows its **Lifecycle** button in a warning color. The dialog names the rule, with **View rule** to open it.

## What's next

- [Age indicators with decay rules](/opencti/configure/decay/), to tune lifetimes and reaction points to your detection thresholds.
- [Feeds and streams](/opencti/use/share/feeds/), to send valid indicators to your detection tools.
- [Explore observations](/opencti/use/explore/observations/), to search indicators by score, validity or detection.
- [Reliability and confidence](/opencti/concepts/reliability-confidence/), to understand how the score differs from confidence.
