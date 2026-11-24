---
title: Get your first quantified risk scenario in eight steps
description: Follow Get started, the eight steps that take a new OpenCRQ tenant to its first quantified risk scenario, and read on its overview which steps are done and which are left.
sidebar_position: 1
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

:menu[Get started] takes a new tenant to its first quantified risk scenario in eight steps. Each step fills one part of what a price rests on: who you are, what an attacker can reach, who attacks, what defends, what a loss costs, what loss you accept, what you protect, and the scenario itself. Its overview shows which steps are done and which are left, so you can stop and come back where you left off.

The steps fill the risk scope every new tenant gets, **Enterprise**, measured over the whole estate. If risk scopes are new to you, read [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/) first.

## Before you start

- You are a member of the tenant. Opening the steps and answering them needs no capability. Setting up a connector from a step, in **Assets**, **Threats** or **Defenses**, needs **Manage Security**.
- The tenant has Get started. Every tenant created from the platform gets it, and so does the tenant OpenCRQ creates at its first boot. A tenant without it shows no **Get started** in its menu, and the address of Get started opens **Home**.

## The eight steps

| Step | What it sets | Read more |
| --- | --- | --- |
| **Company** | The scope's name, sector, country and size, which set how often companies like yours are attacked | [Set your company's sector, country and size](/opencrq/use/get-started/company/) |
| **Assets** | The size of your estate and the assets an attacker can get in through | [Add your assets and declare the size of your estate](/opencrq/use/get-started/assets/) |
| **Threats** | The sources of the threats that target your sector and your country | [Import the threats that target your company](/opencrq/use/get-started/threats/) |
| **Defenses** | How strong your controls are: "A control left unassessed counts as absent, so your losses are overestimated." | [Set how strong your controls are in the Defenses step](/opencrq/use/get-started/defenses/) |
| **Finances** | The financial profile, which gives each loss its amount | [Give your financial figures in the Finances step](/opencrq/use/get-started/finances/) |
| **Tolerance** | The yearly loss you accept, and how likely you accept to exceed it | [Set the yearly loss your company accepts](/opencrq/use/get-started/tolerance/) |
| **Protect** | The primary assets: what an attack can reach and what it would cost you | [Choose what you protect and link the assets behind it](/opencrq/use/get-started/protect/) |
| **Scenarios** | Your first scenarios, each an attack outcome paired with what it reaches | [Create and price your first risk scenarios](/opencrq/use/get-started/scenarios/) |

You can take the steps in any order, and leave one for later.

## Find Get started

- **In the menu.** While the steps are unfinished, :menu[Get started] comes first, above **Home**. With the menu expanded, it shows a bar and how many steps are done, such as 2 of 8. Once the steps are finished, it moves to the end of the menu, after **Settings**, without the bar.
- **From the tenant's home.** While the steps are unfinished, the tenant's home, which the OpenCRQ logo opens, sends you to Get started. It does so until you have opened Get started once in this browser: after that, it opens **Home**. The browser remembers it for each user and tenant, so another browser opens Get started again.

## Go through the steps

:::steps
1. Open :menu[Get started]. The overview reads "Get a first quantified risk scenario in a few minutes." and lists the eight steps, each with its status.

   ::screenshot{id=opencrq-get-started-overview alt="The Get started overview of Grand Meridian Hotels: the count of steps done and started above the eight steps from Company to Scenarios, each with its status, such as Already configured, Started or To do, and the Start button."}

2. Select **Start** to open **Company**, the first step, or select the row of the step you want.
3. Answer the step. A **Why this step?** card says what it changes in the price, and the stepper above shows the eight steps: select one to go to it.
4. Select **Next** to go on to the following step, or **Back** to return to the one before. **Back** on the first step returns to the overview.
5. On **Scenarios**, the last step, select **Next**. Once the steps are finished, it opens **Your platform is set up**: see [Reach the end of Get started and open your first risk scenarios](/opencrq/use/get-started/done/). Otherwise, it returns to the overview.
:::

## Read where each step stands

Above the steps, the overview counts them: "2 of 8 steps done", followed by "· 1 started" when some are started. Each row shows its status:

| Status | What it means |
| --- | --- |
| **Already configured** | The step is done. A check replaces its number, in the overview and in the stepper |
| **Started** | Part of the step is answered. A ring around its number fills as you go, and a line says how far: the kinds of asset declared for **Assets**, such as "5 of 12 kinds declared", the capabilities assessed for **Defenses**, and the questions answered for **Finances** |
| **To do** | Nothing is answered yet |
| **Syncing** | An import runs behind the step: Microsoft Defender for **Assets**, OpenCTI or OpenCRQ Default Threats for **Threats**. The row names the source and what it imported so far, such as "OpenCTI · 42 threats imported", with its progress in percent once the import can tell |

No status is set by hand: each one reads what you can see of the tenant, wherever you set it. Threats and assets follow your markings, and the threats streamed from OpenCTI your access in OpenCTI, so two members can see **Assets** or **Threats** at a different status. A risk tolerance set from the scope's **Details** checks **Tolerance** as well as the step does.

| Step | Started when | Done when |
| --- | --- | --- |
| **Company** | | The scope has a sector and a country |
| **Assets** | A kind of asset is declared, or you can see an asset of the tenant | The 12 kinds of the declared estate are answered, **None** included |
| **Threats** | | You can see a threat of the tenant, from any source |
| **Defenses** | Some capabilities of the control catalogue are assessed | Every capability is assessed, for the tenant or for the scope |
| **Finances** | Some questions of the financial profile are answered | Every question is answered |
| **Tolerance** | | The scope has a risk tolerance |
| **Protect** | | The scope has a primary asset |
| **Scenarios** | | A scenario of the scope is priced by its latest simulation |

## When Get started is finished

Get started is finished once no step reads **To do**: a started step does not hold it back. Then:

- The button under the steps reads **What you can do now** instead of **Start**. It opens **Your platform is set up**, with **See your risk scenarios** and five next tasks: **Risk scenarios**, **Assets**, **Treatment plans**, **Budget reviews** and **Compliance**.
- :menu[Get started] moves to the end of the menu, and the tenant's home opens **Home**.

Since the statuses read the data you can see, a step can go back to **To do**: delete the scope's last primary asset, and **Protect** is to do again. Get started is then unfinished, and comes first in the menu again.

:::example[Pick up where Grand Meridian left off]
Claire Morel, the risk manager of Grand Meridian Hotels, spent her first hour on Get started. She described the company, declared five kinds of asset, and the OpenCTI stream she bound brings the threats aimed at the hospitality sector. The next morning, she opens :menu[Get started]:

- The overview reads "2 of 8 steps done · 1 started".
- **Company** and **Threats** read **Already configured**.
- **Assets** reads **Started**, "5 of 12 kinds declared": the seven other kinds can wait.
- **Defenses**, **Finances**, **Tolerance**, **Protect** and **Scenarios** read **To do**.

She selects the **Defenses** row and goes on from there. Once **Scenarios** prices a first scenario and no step reads **To do**, Get started is finished, even with **Assets** still started.
:::

## Check that it works

- The menu entry counts the steps done, and the overview counts the steps done and started.
- Once every step is done or started, **What you can do now** replaces **Start**, and :menu[Get started] comes last in the menu.

## What's next

- [Set your company's sector, country and size](/opencrq/use/get-started/company/), the first step.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), to understand what each step changes in the price.
- [Run several tenants in one platform](/opencrq/administer/tenants/), for what a new tenant starts with.
