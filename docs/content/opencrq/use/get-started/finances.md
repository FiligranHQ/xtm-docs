---
title: Give your financial figures in the Finances step
description: Answer the financial questionnaire of your company in the Finances step of Get started, so OpenCRQ turns each incident it simulates into a loss.
sidebar_position: 6
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Finances** is the fifth step of :menu[Get started]. Your financial figures turn an incident into a loss: downtime, notification, fines, ransom. The step asks the 21 questions of the financial profile of the risk scope Get started fills, **Enterprise** unless you renamed it on the **Company** step. Each scenario's loss estimates derive from these answers, and every extra answer sharpens one.

## Before you start

- You are a member of the tenant.
- Have the figures at hand: revenue and gross margin from finance, recovery targets from IT, the number of records and of card transactions from the data protection officer, the liability clauses from legal. Bands are enough for most answers.
- Choose the **Country** on the [Company step](/opencrq/use/get-started/company/) first: the step reads the labour cost benchmark of that country.

## What the step asks

Under **Why this step?**, the questionnaire shows a progress bar and how many answers are given, such as **5 of 21 answered**. Its eight sections are all open, each with its own count: **A · Revenue and margin**, **B · Digital dependency**, **C · Data held**, **D · Regulatory**, **E · Customers and contracts**, **F · Funds and IP**, **G · Extortion** and **H · Loss history**. They are the sections of the financial profile, and [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/#answer-the-financial-profile) says what each one asks and which loss drivers it sets.

Section A opens on **Loaded daily labour cost**, which is not one of the 21 questions. It starts empty. Left blank, OpenCRQ uses the benchmark of the scope's country, such as 480 USD / day for France, or the platform default of 300 USD / day, and flags internal response labour as low confidence. The field says which one applies.

The step is started once one question is answered, and done once the 21 are. A started step is enough for Get started to finish.

## Answer the questions

:::steps
1. Open :menu[Get started] and select **Finances** in the stepper, or select **Next** on the **Defenses** step.
2. Answer the sections, in any order. A number you type is saved when you leave its field; a choice is saved as soon as you click it.

   ::screenshot{id=opencrq-get-started-finances alt="The Finances step of Get started for Grand Meridian Hotels: the Why this step? card, the questionnaire counting its first answer, and section A open on an empty loaded daily labour cost and an annual revenue of 830,000,000 USD."}

3. Leave blank what you do not know yet: you can come back to the step at any time.
4. Select **Next** to go on to the **Tolerance** step.
:::

The step has no save button, and no message confirms a save. Answers save together: each save checks the whole questionnaire. An answer that is not valid shows why under its field, and no answer saves until you correct it. When a save fails, an error message says so, and your next change tries again. When the answers cannot be read, the step reads "Your financial figures could not be loaded." and shows no question, so that nothing is written over the answers already saved.

Each save is the same as in the scope's **Financial profile** tab: OpenCRQ recomputes the loss estimates of every scenario of the scope, then prices again those whose amounts moved. A new or changed **Annual revenue** also sets the scope's revenue band, the one the **Company** step shows. When the band moves, OpenCRQ computes again the threat event frequency of the scenarios set to **Auto**; the band changes it only when the scope has no **Employee Count**.

:::example[Give Grand Meridian's figures]
The **Enterprise** scope of Grand Meridian Hotels covers the whole chain, France and Spain. Claire Morel opens the **Finances** step with the figures of the finance team:

- **Annual revenue**: `830,000,000` USD, $620 million in France and $210 million in Spain. The scope's revenue band follows, **$100M-$1B**. The **Employee Count** of the **Company** step already sizes the chain, so the band leaves the threat event frequency as it is.
- **Gross margin**: `31` %.
- **Operating days per year**: **365 (continuous)**, for hotels open every day.
- **Regulatory regime**: **NIS2 important**.
- **Ransom payment posture**: **No**, which records the extortion payment as zero rather than leaving it unestimated.

She leaves **Loaded daily labour cost** blank: the field reads the 480 USD / day benchmark for France. The step now reads **5 of 21 answered** and shows as started in the stepper. She will answer the data held with the data protection officer, and selects **Next**.
:::

## Check that it works

- The step counts your answers, and each section its own.
- The **Financial profile** tab of the **Enterprise** scope, under :menu[Details], shows the same answers.
- The stepper shows **Finances** started, then a check once the 21 questions are answered.

## What's next

- [Set the yearly loss your company accepts](/opencrq/use/get-started/tolerance/), the **Tolerance** step that **Next** opens.
- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/), to read the loss estimates of a scenario and correct a driver by hand.
- [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/), the other half of the price.
- [Set how strong your controls are in the Defenses step](/opencrq/use/get-started/defenses/), the step before.
