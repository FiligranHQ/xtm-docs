---
title: Get notified of score drops and changes
description: Set triggers that alert you in OpenAEV, by email or through a webhook when a scenario's score drops or a simulation, finding or asset changes, alone or as a digest.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, exercise-planner, admin]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/notifications/
---

OpenAEV tells you when something you follow happens, so you do not have to check every scenario after each run. A scenario scores lower than its previous simulation, a simulation of your exercise is updated, a finding appears. You set what to watch in triggers; each notification reaches you in the **Notification center**, by email, or in another tool through a webhook. This page subscribes to a scenario in one click, creates triggers and digests, reads the alerts, and, for administrators, sets the notifiers.

## Before you start

- Triggers are personal: every user creates their own, and they notify only them. To watch scenarios, simulations, injects, teams, players or vulnerabilities, no capability is needed; to watch findings, assets, payloads or the other types, your role must let you list them. A trigger only tells you about what you can read: see [Who gets notified](#who-gets-notified).
- To subscribe from a scenario's page, your role has **Manage assessment**, or your group holds the scenario's **Manage+Delete** grant.
- The notifiers you can choose are set by an administrator, with **Manage tenant settings**: see [Set the notifiers](#set-the-notifiers). Every tenant has two built-in notifiers: **User interface** and **Default mailer**.

## How a notification is made

::diagram{id=openaev-notifications-flow}

A **live trigger** watches one **Resource type**, such as **Scenario** or **Finding**, for the events you choose: **Create**, **Update**, **Delete**, for a scenario **Score degradation**, and for a simulation **Critical asset reached**. Each matching event sends a notification through the trigger's notifiers at once. A trigger with **Filters** is not told of deletions, except on scenarios and simulations.

A **regular digest** watches nothing itself: it gathers what some of your live triggers caught, and sends it as one notification every hour, day, week or month. A period without any event sends nothing.

The notifier decides where the notification goes:

| Notifier | Where the notification goes |
| --- | --- |
| **User interface** | The **Alerts** tab of your **Notification center**, and the dot on the bell of the top bar |
| An **Email** notifier, such as **Default mailer** | Your email address, through the platform's mail server |
| A **Webhook** notifier | An HTTP request to another tool, such as a chat or a ticketing system |

Only a trigger with the **User interface** notifier fills the **Alerts** tab.

### When a score counts as degraded

When a simulation of a time-based scenario ends on its own, OpenAEV compares its results with those of the previous finished simulation of the same scenario. A chained scenario raises no **Score degradation**. OpenAEV waits one hour by default, so that expectations have time to be scored. A lower score of any technical expectation, prevention, detection, vulnerability or logging, raises **Score degradation**. Its message shows the prevention and detection scores, and the logging one when the simulations have logging expectations. It reads, for example, `[scenario] Velvet Harbor replay: score degradation detected - prevention 80% -> 60%, detection 70% -> 70%`, followed by the dates of the two simulations; the **Alerts** tab shows it without the `[scenario]` prefix. The players' expectations, such as reading an article or completing a challenge, are not compared, nor results still pending. The delay is `openaev.notification.simulation-completed-delay-seconds`, in the [configuration reference](/reference/openaev/configuration/#openaev-notification).

### When a simulation reaches a critical asset

**Critical asset reached** concerns chained simulations, the ones that draw an attack path. When a chained simulation finishes on its own, OpenAEV reads its attack path. If an endpoint of the path is an asset whose **Criticality** is **High** or **Very high**, the event fires once for the simulation. A simulation you stop yourself raises nothing, nor a time-based one, which draws no attack path. The message names the simulation and the attack path, never the asset: `[simulation] Velvet Harbor chained run: attack path reached a critical asset`, followed by the address of the attack path in the API. Set the criticality of your assets as [Add the assets your simulations target](/openaev/use/targets/assets/) explains, and read the path as [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/) shows.

### Who gets notified

A trigger notifies you only of what you can read. For each event, OpenAEV checks that your role, or a grant of your group, lets you read the object, as the rest of the interface does: an event on a scenario you cannot open never reaches you, in the interface, by email or by webhook. A digest checks each event again when it is sent.

When you lose access to a scenario or a simulation, because an administrator removed a grant of your group, removed you from the group, or deleted the group, a trigger that watches **Delete** on it sends you a **Deletion**: `Restricted is no longer visible`, without the object's name.

## Subscribe to a scenario in one click

:::steps

1. Open the scenario and click the bell with a plus sign in its header, **Subscribe to notifications on this entity**.
2. OpenAEV creates a live trigger named **Instance trigger:** followed by the scenario's name. It watches this scenario only, for every event, **Score degradation** included, and notifies you in the **User interface**.

:::

The bell then shows a badge. Click it again, **Manage notifications on this entity**, to change the trigger, for example to add **Default mailer**, or to delete it and unsubscribe.

## Create a live trigger

:::steps

1. Click the bell of the top bar, **Notifications**, then the **Triggers** tab of the **Notification center**.
2. Click **Create Live trigger**.
3. Fill in the **Name**: it shows on every notification the trigger sends.
4. Choose the **Resource type**: **Scenario**, **Simulation**, **Inject**, **Finding**, **Asset**, **Asset group**, **Team**, **Player**, **Payload**, **Vulnerability**, **Security platform**, **Document** or **Challenge**.
5. Check the events to watch: **Create**, **Update**, **Delete**, **Score degradation** for a scenario, and **Critical asset reached** for a simulation. Choose at least one.
6. Optionally, add **Filters** to watch only some of these resources, for example the simulations with a given tag. Without filters, the trigger watches them all. With filters, it misses **Delete**, except on scenarios and simulations.
7. Choose one or more **Notifiers**.
8. Click **Create**.

:::

On a **Finding**, a filter on the value of a sensitive finding, such as a credential, matches its masked form only.

## Group alerts in a digest

:::steps

1. On the **Triggers** tab, click **Create Regular digest**.
2. Fill in the **Name**.
3. Under **Composed triggers**, choose the live triggers to gather. A digest cannot contain another digest.
4. Choose the **Period**: **Hour**, **Day**, **Week** or **Month**. For a week, choose the **Day of week**; for a month, the **Day of month**. Except for **Hour**, set the **Time (UTC)**.
5. Choose one or more **Notifiers**.
6. Click **Create**.

:::

A live trigger in a digest still notifies on its own. To receive its events only in the digest, give the live trigger the **User interface** notifier alone. A disabled live trigger records nothing, so the digest gets nothing from it either.

## Manage your triggers

The **Triggers** tab lists your triggers with their **Name**, **Type**, **Resource type**, **Events**, and whether they are **Enabled**. The menu of a line offers **Update**, **Disable** or **Enable** to pause a trigger without losing it, and **Delete**.

::screenshot{id=openaev-notifications-triggers alt="The Triggers tab of the Notification center: the live triggers Velvet Harbor replay score drop on Scenario for Score degradation and Front-desk simulations on Simulation for Create and Update, and the digest Monday validation summary composing 2 triggers every week, all three Enabled."}

## Read your alerts

The **Alerts** tab lists what your triggers sent to the **User interface**, the unread ones marked with a dot. Each line gives:

- **Operation**: **Creation**, **Modification**, **Deletion**, **Score degradation**, **Critical asset reached**, or **Multiple** for a digest.
- **Message**: what happened, without the resource type in brackets that starts the message, such as `[simulation]`, or for a digest, the number of events it holds. Hover a message cut short to read it whole.
- **Original creation date**: when the notification reached the interface, so for a digest, when it was sent.
- **Trigger name**: click it to list only that trigger's alerts.

Click a line to open what it is about, which also marks it as read: the scenario whose score dropped, the finding that appeared, the simulation that reached a critical asset. A payload opens in the **Threat Arsenal**. A deletion opens nothing, nor a vulnerability, document or challenge. Click a digest to list its events in **Digest details**, each one clickable.

**Mark all as read** clears the dots. Select lines to mark them as read or unread, or to delete them, together.

::screenshot{id=openaev-notifications-alerts alt="The Alerts tab of the Notification center: the unread alerts of the trigger Front-desk simulations, a Creation for each new simulation and a Modification for each change, with their message and original creation date."}

## Set the notifiers

An administrator with **Manage tenant settings** adds email and webhook notifiers for the whole tenant. Users then choose them in their triggers.

:::steps

1. Open :menu[Settings > Customization > Notifiers] and click **Create**.
2. Fill in the **Name**, as users will choose it, and a **Description**.
3. Choose the **Type**: **Email** or **Webhook**.
4. For an email, fill in the **Subject template** if the default, the trigger's name after `[OpenAEV]`, does not suit you. Fill in the **Body template (FreeMarker, empty = default template)** if the default body does not suit you.
5. For a webhook, fill in the **URL**, which starts with `https://` or `http://`, and the **Verb**, **POST** by default. Add the **Headers** the receiving tool expects, and the **Body template (FreeMarker, empty = default JSON payload)**.
6. Click **Create**.

:::

::screenshot{id=openaev-notifications-notifiers alt="Settings, Customization, Notifiers: the two built-in notifiers, User interface and Default mailer, with the Create button."}

Templates use the variables `${notification_name}`, the trigger's name, `${notification_type}`, `${platform_url}`, `${user_email}`, `${user_name}` and `${content}`, the events as JSON. An email notifier writes to the trigger owner's own address. A webhook to an internal or private address is refused, unless the platform allows it with `openaev.notification.webhook-allow-internal-targets`: see the [configuration reference](/reference/openaev/configuration/#openaev-notification).

:::example[Be told when the Velvet Harbor replay scores lower]
Example CERT replays Velvet Harbor against the front-desk workstations of Grand Meridian Hotels every week, after each change of the Meridian EDR policy. The validation engineer wants an email the day the replay scores lower, and a Monday summary of the week. They create a live trigger:

| Field | Value |
| --- | --- |
| **Name** | `Velvet Harbor replay score drop` |
| **Resource type** | **Scenario**, every scenario of the tenant: no filter |
| Events | **Score degradation** |
| **Notifiers** | **Default mailer**, **User interface** |

A second live trigger, `Front-desk simulations`, watches **Simulation** for **Create** and **Update**, with **User interface** alone. Then a regular digest, `Monday validation summary`, composes both, with **Period** **Week**, **Day of week** **Monday**, **Time (UTC)** `07:00`, and **Default mailer**.

On 16 January 2026, the replay after the EDR update scores 60% in prevention instead of 80%. One hour later, the engineer receives the email; the Monday summary repeats it with the four simulations of the week. Meanwhile, the exercise planner of the crisis exercise subscribes to the scenario **Velvet Harbor crisis night** with its bell, and sees each change of its timeline in their **Alerts**.

The engineer also runs a chained scenario against the booking servers, whose database server has the **Criticality** **Very high**. A third live trigger, `Booking database reached`, watches **Simulation** for **Critical asset reached**, with **Default mailer**: the first chained run whose attack path gets to that server sends one email, and the engineer opens the attack path to see which step led there.
:::

## What's next

- [Run a simulation](/openaev/use/simulations/) and [Simulation reports](/openaev/use/simulations/reports/).
- [Findings](/openaev/use/findings/), which triggers can watch.
- The [configuration reference](/reference/openaev/configuration/#openaev-notification) for the notification settings.
