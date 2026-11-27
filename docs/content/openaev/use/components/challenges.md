---
title: Give your players challenges to solve
description: Ask your players a question they must answer from the evidence of the exercise, check their answer against flags, and score it in the simulation's results.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/components/challenges/
---

A challenge is a question your players answer during the exercise, in the manner of a capture the flag (CTF). Find the account the attacker used, the domain the malware called, the time the data left. You give them the evidence, they type an answer, and OpenAEV checks it against the flags you set. The result counts in the simulation, player by player. This page creates a challenge, publishes it with an inject, and reads the results.

## Before you start

- Your role has **Manage challenges** to create and change challenges, and **Delete challenges** to delete them.
- To publish one, your role has **Manage assessment** on the scenario or simulation, or your group holds its **Manage+Delete** grant.
- The players are in teams with an email address: the challenge reaches them by email. See [People](/openaev/use/targets/people/).

## How a challenge is answered

::diagram{id=openaev-challenges-flow}

A challenge belongs to the platform, not to a scenario: several scenarios can publish the same one. In :menu[Components > Challenges], the **Simulations** column shows up to three simulations whose injects publish the challenge, each a link to the simulation. The inject that publishes it sends each player an email with their own link to the challenges page. There, the player opens the challenge's card, reads it, downloads its documents and submits a flag.

## Create a challenge

:::steps

1. Open :menu[Components > Challenges] and click **Create a new challenge**.
2. Fill in the **Name**, which the players read on the card.
3. Optionally, fill in the **Category**: the players' page groups the challenges by category, such as Investigation or Communication.
4. Write the **Content**: the situation and the question, as the players read it.
5. Optionally, set the **Score** the card shows the players, and the **Max number of attempts**. Leave the maximum empty to allow any number.
6. Under **Documents**, add the evidence the players need, such as an alert export. See [Share files with your players](/openaev/use/components/documents/).
7. Under **Flags**, choose the **Flag type** and type the **Value** of an answer you accept. Click **Add** for another accepted answer. A challenge needs at least one flag.
8. Click **Create**.

:::

The **Flag type** decides how an answer is compared with the **Value**:

| Flag type | An answer is correct when | Example value |
| --- | --- | --- |
| **Text** | It is the value, whatever its case | `night.audit` accepts `Night.Audit` |
| **Text (case-sensitive)** | It is the value, letter for letter | `T1566.001` |
| **Regular expression** | The whole answer matches the expression | `(?i)(meridian\\)?night\.audit` accepts `MERIDIAN\night.audit` |

An answer is correct as soon as it matches one flag. Spaces count: `night.audit ` with a trailing space does not match a **Text** flag.

::screenshot{id=openaev-challenges-create alt="The Create a new challenge panel: name, category, content, score, maximum attempts, documents, tags and the flags that check an answer."}

## Publish it with an inject

:::steps

1. Open the **Injects** tab of the scenario or simulation and click **Create**, then choose the inject type **Publish challenges**.
2. Give the inject a **Title** and set, under **Scheduling**, when the challenge comes out.
3. Under **Targeted teams**, choose the teams who must answer, or turn on **All teams**.
4. Under **Challenges to publish**, click **Add challenges**, select one or more challenges, and click **Add**.
5. Under **Inject data**, change the **Subject** and the **Body** of the email if you wish. The default body lists the challenges with each player's link.
6. Under **Inject expectations**, keep **Expect targets to complete the challenge(s)**.
7. Click **Create**.

:::

Once an inject uses a challenge, the header of the scenario or simulation counts its **Challenges**, and its trophy icon, **Preview challenges page**, opens the players' page. In preview mode, attempts are not counted and results are not recorded.

## What the players see, and how they score

The link of the email opens the player's challenges page, grouped by **Category**. Each card shows the challenge's **Score**, its documents, and the attempts used out of the maximum. The player opens a card, types the answer in **Flag** and clicks **Submit**:

- **Flag is correct! It has been successfully submitted.** The player's expectation is met, with the full score of the expectation, 100 by default.
- **Flag is not correct! Try again...** The attempt counts.
- **Max attempts exceeded.** The player used all their attempts: their expectation fails, and the card takes no more answers.

The **Score** of the challenge is what the card displays; the result in the simulation is the score of the inject's expectation. A player who never answers fails the expectation when it expires: after its **Expiration time**, 24 hours by default (`openaev.expectation.challenge.expiration-time` in the [configuration reference](/reference/openaev/configuration/#openaev-expectation)). Read the results on the inject: see [Expectations](/openaev/use/expectations/).

:::example[Find the account that ran the loader]
During the crisis exercise of Grand Meridian Hotels, on the night of 9 January 2026, Example CERT wants to know whether the crisis cell can read the alert of its own EDR. The exercise planner creates the challenge:

| Field | Value |
| --- | --- |
| **Name** | `Find the account that was used` |
| **Category** | `Investigation` |
| **Content** | The EDR alert of the night shows a PowerShell process started from a booking form. Which account ran it? |
| **Score** | 50 |
| **Max number of attempts** | 3 |
| **Documents** | `edr-alert-front-desk.pdf` |
| **Flags** | **Text** `night.audit`, and **Regular expression** `(?i)(meridian\\)?night\.audit` |

The inject `Which account was used?`, of type **Publish challenges**, 50 minutes after the start, targets the team **Crisis cell**. Hélène Marchetti first answers `paul.garnier`, the night auditor on shift, then reads the alert again and submits `MERIDIAN\night.audit`: correct at the second attempt. The shared front-desk account becomes the first decision of the debrief.
:::

::screenshot{id=openaev-challenges-preview alt="The challenges page of Velvet Harbor crisis night as players see it: Find the account that was used, in Investigation, worth 50 points, with 3 attempts and one document."}

## What's next

- [Collect lessons learned after an exercise](/openaev/use/components/lessons/).
- [Publish fake press articles during an exercise](/openaev/use/components/media-pressure/).
- [Expectations](/openaev/use/expectations/), to read and adjust the players' results.
