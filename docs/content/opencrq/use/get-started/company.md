---
title: Set your company's sector, country and size
description: Name your company, its sector, its country and its size in the first step of Get started, so OpenCRQ reads how often companies like yours are attacked.
sidebar_position: 2
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

:menu[Get started] leads a tenant to a first quantified risk scenario in eight steps: **Company**, **Assets**, **Threats**, **Defenses**, **Finances**, **Tolerance**, **Protect** and **Scenarios**. The first step, **Company**, describes your organization. Its sector, its country and its size set how often an attack is expected each year: the threat event frequency of your scenarios.

They also weigh the threats of the [Threat Center](/opencrq/use/threats/threat-center/). A threat aimed at your country counts 2.5 times, one aimed at your sector 4 times, one aimed at both 10 times.

Every new tenant, the **Default** tenant included, comes with Get started and a risk scope named **Enterprise**, measured over the whole estate. The steps fill that scope: the **Company** step edits its profile.

## Before you start

- You are a member of the tenant.
- You know how many people your company employs. Its annual revenue comes later, in the **Finances** step.

## What the step sets

| Field | What to enter | What it changes |
| --- | --- | --- |
| **Name** | The name of the risk scope. It reads **Enterprise** until you change it. It is required: an empty name reads **A name is required** | The name the scope shows everywhere |
| **Sector** | Your industry, from **Select a sector** | The threat event frequency reads the attack rates measured for your sector. Left blank, or on a sector without measures, it reads every sector |
| **Country** | Where you operate, from **Select a country** | The same for your country. Left blank, or on a country without measures, it reads every country |
| **Employee Count** | A whole number, 0 or more | The size of your company, read before the revenue band |
| **Revenue band** | The band of your annual revenue. The field says "Derived from the annual revenue you can set in the Finances step." | The size of your company when no employee count is entered. The **Annual revenue** of the **Finances** step sets it each time you enter or change it. Without either, every size counts alike |

## Describe your company

:::steps
1. Open :menu[Get started]. It comes first in the menu while the steps are unfinished. Until then, the tenant's home also opens it the first time you visit the tenant in a browser.
2. Select **Start**, or the **Company** row. The step opens under the stepper of the eight steps, with **Why this step?** above the form.
3. Check the **Name**. Keep **Enterprise** if the scope covers your whole company.
4. Choose the **Sector** and the **Country**.
5. Enter the **Employee Count**.
6. Choose the **Revenue band**, or leave it to the **Finances** step.

   ::screenshot{id=opencrq-get-started-company alt="The Company step of Get started for Grand Meridian Hotels: the stepper with Company first, the Why this step? card, then Name on Enterprise, Sector on Hospitality and Country on France."}

7. Select **Next** to go on to the **Assets** step.
:::

The step has no save button. A list saves your choice at once; **Name** and **Employee Count** save when you leave the field. No message confirms it, and a field you leave unchanged writes nothing. **Back** returns to the overview of the steps, and **See the risk scope** opens the scope's **Details**.

When the sector, the country, the employee count or the revenue band changes, OpenCRQ computes again the threat event frequency of the scenarios set to **Auto**. It then prices them again.

:::example[Describe Grand Meridian Hotels]
Claire Morel, the risk manager of Grand Meridian Hotels, opens :menu[Get started] on the tenant's first day and selects **Start**:

- She keeps **Name** on **Enterprise**: the scope covers the whole chain, and she creates the France and Spain scopes later.
- **Sector** **Hospitality** and **Country** **France**, where the head office runs the booking platform.
- **Employee Count** 5800: 4,200 people in France and 1,600 in Spain. The chain falls in the class of 5,000 employees or more.
- She leaves **Revenue band** to the **Finances** step. There, the annual revenue of $830 million sets it to **$100M-$1B**.

The **Company** step now shows its check in the stepper, and she selects **Next**.
:::

## Check that it works

- The stepper shows a check on **Company** once both **Sector** and **Country** are set.
- On the overview of the steps, the **Company** row reads **Already configured**. The other rows read **Started**, **To do** or **Syncing**.
- In the risk scope, :menu[Threat Event Frequency] shows the country, the sector and the size in its **Profile**.

## What's next

- [Add your assets and declare the size of your estate](/opencrq/use/get-started/assets/), the next step.
- [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/) of a scenario, from the profile or by hand.
- [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/), to understand the scope the steps fill.
