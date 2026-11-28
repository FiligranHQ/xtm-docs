---
title: Collect lessons learned after an exercise
description: Build the debrief questionnaire of a crisis exercise from a template, send it to the teams that played, and read their answers and your objectives in one place.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/components/lessons/
---

The lessons learned questionnaire is the debrief of an exercise, asked to the people who played it. How fast did the alert reach you? Did the press article change your decisions? What would you do differently? You prepare the questions in categories and send them by email to the teams that played. Every answer and comment comes back to the simulation, next to the objectives your control team evaluates. Templates let you reuse the same questions from one exercise to the next. This page turns the module on, builds the questionnaire, sends it and reads the answers.

## Before you start

- Your role has **Manage assessment**, or your group has a **Manage+Delete** grant on the scenario or simulation.
- To create the questionnaire's categories, and to manage templates in :menu[Settings > Customization > Lessons learned], your role has **Manage lessons learned**. To apply a template, **Access lessons learned** is enough.
- The players are in teams, each with an email address. See [People](/openaev/use/targets/people/).

## How the questionnaire works

::diagram{id=openaev-lessons-flow}

The questionnaire is made of categories, each with its questions and the teams it targets. A player answers the categories that target one of their teams. For each question, they give a **Global evaluation** from 0 to 100, and optionally say **What worked well** and **What didn't work well**. They submit once, and cannot change their answers afterwards.

You can prepare the questionnaire in the scenario: each simulation created from it copies its categories, questions, targeted teams and objectives. Only a simulation sends the questionnaire and collects answers.

## Turn the module on

A new scenario or simulation has no lessons learned.

:::steps

1. Open the scenario or the simulation, open the menu of its header, and choose **Update**.
2. Under **Modules**, turn on **Enable lessons learned**.
3. Click **Update**. The **Lessons learned** tab appears.

:::

A scenario or simulation that already had categories, objectives or anonymized answers before the module could be turned off keeps it on.

## Prepare a template

A template holds categories and questions you reuse in every exercise. It is optional: you can write the questions in the exercise directly.

:::steps

1. Open :menu[Settings > Customization > Lessons learned] and click **Create**. Give the template a **Name** and a **Description**, then click **Create**.
2. Open the template and click **Add a category**. Fill in its **Name**, a **Description**, and its **Order**, the position among the categories, then click **Create**.
3. In the category, click **Create a new lessons learned question**. Write the **Content**, the question itself, an **Explanation** that helps the player answer, and its **Order** in the category, then click **Create**.
4. Repeat for every category and question.

:::

::screenshot{id=openaev-lessons-template alt="The lessons learned template Crisis exercise debrief: the categories Alert and escalation and Communication, each with its question."}

## Build the questionnaire of the exercise

:::steps

1. Open the **Lessons learned** tab of the scenario or simulation.
2. Under **Parameters**, next to **Template**, click **Apply**, choose a template, and click **Apply**. Its categories and questions are added to those already there. To write your own instead, click **Create** next to **Categories and questions**.
3. In each category, under **Targeted teams**, add the teams that must answer it. A category without a team reaches nobody.
4. To show the answers without the players' names, turn on **Anonymize answers**, under **Questionnaire mode**, and confirm. You cannot turn it off afterwards.
5. Under **Check**, click **Preview** to read the questionnaire as a player will.

:::

A category you create in the exercise takes a **Name** and an **Order**; a question, its **Content** and an **Order**. The order is a whole number, 0 or more: otherwise the form shows **Must be a whole number** or **Must be greater than or equal to 0**, and **Create** stays disabled.

**Clear out**, next to **Categories and questions**, deletes every category and question of the exercise, to start again.

## Send the questionnaire

When the simulation is over:

:::steps

1. On the **Lessons learned** tab of the simulation, under **Control**, click **Send** next to **Questionnaire**. It stays disabled until a category targets a team with players.
2. Check the **Subject** and the **Message**. The message holds `${lessons_uri}`, which becomes each player's own link: keep it. A message with no visible text shows **This field is required.**
3. Click **Send**.

:::

Each player of the targeted teams receives one email, with a link that opens their questionnaire without signing in. **Reset**, next to **Answers**, deletes every answer received, for example after a dry run with your control team.

## Read the answers

The **Lessons learned** tab of the simulation brings together:

- At the top, the **Overall objectives score**, the number of **Simulation logs**, the **Poll replies** received and the **Messages** exchanged.
- **Objectives**: the goals your control team sets for the exercise, each with a **Title**, a **Description** and a **Priority**, a whole number, 0 or more. Objectives are listed by priority, the lowest number first. Each evaluator scores an objective from 10 to 100. An objective's score is the average of its evaluations, and **Overall objectives score** is the average of the objectives.
- **Crisis intensity (injects by hour)**: the number of injects the players received, hour by hour, to compare the answers with the pressure they were under.
- **Categories and questions**: under **Results**, each question shows its number of answers, how many of them contain comments, and the average evaluation. Click a question to read every answer with its comments, under the player's name or **Anonymized**.

:::example[Debrief the Grand Meridian crisis night]
Example CERT played the Velvet Harbor intrusion to the crisis cell of Grand Meridian Hotels on the night of 9 January 2026. Its template **Crisis exercise debrief** holds two categories:

| Category | **Order** | Questions |
| --- | --- | --- |
| Alert and escalation | 1 | Was the crisis cell on the bridge within 30 minutes of the alert? (The playbook asks for 30 minutes.) |
| Communication | 2 | Did the holding statement go out before the press article was shared? (Hotel Business Daily published at 1 h 30.) |

In the simulation **Velvet Harbor crisis night, 9 January 2026**, the planner applies the template and targets the team **Crisis cell** in both categories. The control team sets two objectives: **Crisis cell assembled within 30 minutes of the alert**, and **Holding statement approved before the press publishes**.

On the morning of 10 January, the planner sends the questionnaire. Hélène Marchetti and Sophie Laurent answer; Marc Dupuis does not. **Communication** averages 40: both comments under **What didn't work well** say nobody owned the answer to the journalist. That gap opens the action plan.
:::

## What's next

- [Publish fake press articles during an exercise](/openaev/use/components/media-pressure/) and [Give your players challenges to solve](/openaev/use/components/challenges/), whose results the debrief discusses.
- [Simulation reports](/openaev/use/simulations/reports/), to share the outcome of the exercise.
- [Run a simulation](/openaev/use/simulations/).
