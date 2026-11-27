---
title: Measure your compliance maturity against a framework
description: Rate your organization against NIST CSF 2.0, requirement by requirement, on a maturity scale from 0 to 5, and publish the assessment so its answers feed the controls OpenCRQ prices.
sidebar_position: 2
doc_type: how-to
products: [opencrq]
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

A compliance assessment rates your organization against a framework, requirement by requirement, on a maturity level from 0 to 5. In OpenCRQ it does more than fill a register: once you publish it, its answers feed the strength of the controls each requirement maps to, and the price of your risk scenarios follows. This page creates an assessment on NIST CSF 2.0, answers it, publishes it, and reads the frameworks Drata synchronizes beside it. For the controls themselves, read [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).

## Before you start

- You are a member of the tenant: every member creates, answers, publishes and reopens assessments.
- The framework you can assess in OpenCRQ is **NIST CSF 2.0**, shipped with the platform from NIST's own catalog: 6 functions, 22 categories and 106 subcategories, the requirements you answer. You cannot add a framework.
- To see your ISO/IEC 27001:2022 and SOC 2 readiness from Drata, an administrator with **Manage Security** connects Drata. See [Import compliance readiness from Drata](/opencrq/use/sources/drata/).

An assessment belongs to the tenant, not to a risk scope: it rates the organization as a whole. You can keep several assessments of the same framework, one per quarter for example, and compare them.

## Create an assessment

:::steps
1. Open :menu[Compliance] in the sidebar. The list shows your assessments and the Drata frameworks, with their **Framework**, **Source**, **Progress**, **Maturity**, **Status** and **Updated** date.
2. Select **New assessment**.
3. Enter a **Name**, such as the framework and the period, and keep **NIST CSF 2.0** in **Framework**.
4. Select **Create**. The assessment opens, in **Draft**.

   ::screenshot{id=opencrq-compliance-assessment alt="The NIST CSF 2.0 · Q1 2026 assessment of Grand Meridian Hotels in draft: progress at 3 of 106 requirements, average maturity 2.3 and 20 of 51 controls fed, above the tree of functions, categories and subcategories, each with its owner, due date, controls and maturity."}
:::

## Answer the requirements

The assessment shows the framework as a tree: functions, categories, then the subcategories you answer, each with its **Owner**, **Due date**, the number of **Controls** it feeds and its **Maturity**. **Unanswered** keeps the requirements left to answer; **Feeds controls** keeps the ones that feed an OpenCRQ control, 35 of the 106.

:::steps
1. Select a requirement. Its panel shows the requirement and, under **Controls fed**, the OpenCRQ controls it feeds, or says that it feeds none.
2. Set an **Owner** among the tenant's users and a **Due date** if someone must act on it.
3. Answer **What is the maturity level?** with one level, or select **Not applicable**:

   | Level | Label |
   | --- | --- |
   | 0 | **Nonexistent** |
   | 1 | **Initial** |
   | 2 | **Repeatable** |
   | 3 | **Defined** |
   | 4 | **Managed** |
   | 5 | **Optimized** |

   The information icon beside the question lists the six levels with their definitions.
4. In **Comment**, write what is in place and what is missing: it is the evidence an auditor reads.
5. Select **Save**.
:::

When two people answer the same requirement at the same time, the second save is refused, and the drawer reloads the saved answer with your changes on top. Under a field you both changed, **Saved meanwhile** shows the other value: keep yours or select **Use saved value**, then save again.

The cards at the top follow your answers. **Progress** counts the requirements answered or marked not applicable, out of 106. **Average maturity** is the mean level of the answered requirements, leaving out those not applicable. **Controls fed** counts the controls your answers feed, out of the 51 the framework reaches.

## Publish it, and what it changes

Select **Publish** when the assessment is complete. Publishing locks it: its answers can no longer change, so the next assessment can be compared with it. **Reopen** brings it back to **Draft**.

A published assessment feeds the controls:

- For each requirement, the answer of the latest published assessment that answered it counts: a requirement left unanswered keeps the answer of an earlier published assessment. **Not applicable** feeds nothing, whatever an earlier assessment said.
- A level becomes a strength: **Nonexistent** gives **Not deployed**, **Initial** and **Repeatable** give **Basic**, **Defined** gives **Intermediate**, **Managed** and **Optimized** give **Master**. A control fed by several requirements takes the mean, rounded to the nearest level. When nothing measures the control's coverage, OpenCRQ assumes one from the same answers.
- Drata's reading of a control, when you have one, weighs twice as much as your answers. When Drata reads the highest strength its evidence can prove, higher answers can raise the control above Drata's reading.
- Only the tenant's postures are fed, and only the axes left on **Auto**. A value you chose yourself, and the posture a risk scope declares on its own, stay as they are.

OpenCRQ applies a published assessment to the controls in a background job, at the latest the next day, then prices the scenarios again. Each control lists the answers behind it under **Assessment evidence**. Reopening an assessment stops its answers from feeding the controls.

## Read the Drata frameworks

Once Drata is synchronized, ISO/IEC 27001:2022 and SOC 2 show in the list with **Source** **Drata**, **Status** **Synced** and their readiness as a percentage. They are read-only: open one to see its **Readiness**, its in-scope requirements and controls, and **Open in Drata** to work on it there. Their controls' readiness feeds OpenCRQ's controls beside your assessments.

:::example[Grand Meridian rates its access control after the MFA rollout]
On 2 March 2026, after the front-desk MFA rollout, Claire Morel, risk manager of Grand Meridian Hotels, opens :menu[Compliance], selects **New assessment**, enters **Name** "NIST CSF 2.0 · Q1 2026" and keeps **Framework** **NIST CSF 2.0**.

In the tree, **Feeds controls** narrows the work to the 35 requirements that move the price. PR.AA-03, "Users, services, and hardware are authenticated", feeds ten controls, the identity platform (IDP) among them. She sets herself as **Owner**, the level **Defined**, and the **Comment** "MFA on every account since 2026-02-27; service accounts still on passwords". She rates PR.AA-01 (identities and credentials managed) and PR.AA-05 (access permissions and least privilege) **Repeatable**, with a **Due date** of 30 June 2026.

The assessment, still in **Draft**, reads **Progress** 3 / 106 requirements, **Average maturity** 2.3 and **Controls fed** 20 / 51 controls. Nothing reaches the controls yet: the price only moves once she publishes it.

Once published, the PR.AA-03 answer counts toward a strength of **Intermediate** for the tenant's identity platform, if that axis is on **Auto**. Grand Meridian France declares its own identity platform posture and keeps it; a scope that declares nothing, such as Grand Meridian Spain, inherits the tenant's value, and its scenarios are priced again on it.
:::

## Check that it works

- The assessment shows **Published** in :menu[Compliance], with its **Progress** and **Maturity**.
- A control the assessment feeds lists the answers under **Assessment evidence** on its **Overview**.

## What's next

- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/), and their **Auto** axes.
- [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/) to fund the gaps the assessment shows.
- [Import compliance readiness from Drata](/opencrq/use/sources/drata/).
