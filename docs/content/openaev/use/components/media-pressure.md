---
title: Publish fake press articles during an exercise
description: Write the press articles and social posts of a crisis exercise, publish them to the players at a set time, and see who read them.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/components/media-pressure/
---

Media pressure is what the outside world says about your crisis while your players handle it: a press article that reveals the breach, a post that goes viral, a TV report. You write the articles in the scenario, and an inject publishes them at the time you choose. The players get an email with a link, and read the article on a page that looks like the outlet's. OpenAEV records who read it. This page writes an article, publishes it and checks the reading. The outlet itself, a newspaper, a social network or a TV channel, is a channel: see [Create the media outlets of an exercise](/openaev/use/components/channels/).

## Before you start

- Your role has **Manage assessment**, or your group has a **Manage+Delete** grant on the scenario or simulation.
- The channel exists. To add images or videos to an article, your role also has **Access documents**.
- The scenario plays its injects on a timeline: a chained scenario has no **Configuration** button.
- The platform sends email, and the players are in the teams the inject targets. See [People](/openaev/use/targets/people/).

## How an article reaches the players

::diagram{id=openaev-media-pressure-flow}

An article belongs to the scenario or the simulation it is written in, and only its injects can publish it. A simulation created from the scenario gets its own copy of every article. Before the inject runs, the article is invisible to the players. From the moment it runs, the article is on the channel's page, and stays there for the rest of the simulation.

## Write an article

:::steps

1. Open the scenario or the simulation and click **Configuration** in its header, then open the **Media pressure** tab.
2. Click **Create an article**. The **Create a new media pressure article** dialog opens.
3. Choose the **Channel**, the outlet that publishes the article.
4. Fill in the **Title**, the headline the players read.
5. Fill in the **Author**, as the outlet signs it: a journalist, a newsroom, or the account of a post. Left empty, the players read **Unknown**.
6. Write the **Content**. The editor formats the text as the outlet will show it.
7. Optionally, set **Comments**, **Shares** and **Likes**: the counts the page shows under the article, to make it look viral or quiet.
8. Optionally, under **Documents**, click **Add documents** and choose images for a newspaper, images or videos for a social network, videos for a TV channel. The list only offers the files the channel can show.
9. Click **Create**.

:::

The article appears in the list with its **Name**, **Author**, **Channel** and **Status**. **Not used in the context** means no inject publishes it yet. Click the eye icon, **Preview**, to see the channel's page as the players will.

:::xtm-one[Draft the article with AI]
With XTM One connected and the Enterprise Edition, **Ask AI** beside **Content** offers **Generate an article**. Describe the article in **Input (describe what you want)**, choose a tone, the **Author**, the **Number of paragraphs**, and add **Context**, such as the timeline of your scenario. The same menu fixes the spelling, shortens, lengthens or summarizes a draft. Read every word before the exercise: the players will.
:::

::screenshot{id=openaev-media-pressure-articles alt="The Media pressure tab of the Scenario configuration of Velvet Harbor crisis night: the article Guest records of Grand Meridian Hotels offered on a criminal forum, by the Investigations desk on Hotel Business Daily, not used yet by an inject."}

## Publish it with an inject

:::steps

1. Open the **Injects** tab of the scenario or simulation and click **Create**, then choose the inject type **Publish a media pressure**.
2. Give the inject a **Title** and set, under **Scheduling**, when the article comes out after the start of the simulation.
3. Under **Targeted teams**, choose the teams who learn about the article, or turn on **All teams**.
4. Under **Media pressure to publish**, click **Add media pressure**, select one or more articles, and click **Add**. The status of these articles turns to **Scheduled**.
5. Under **Inject data**, keep **Send email** on to send each player an email with their link to the articles. Change the **Subject** and the **Body** to fit the story; the default body lists the articles with their links.
6. Under **Inject expectations**, keep **Expect targets to read the article(s)** to measure the reading.
7. Click **Create**.

:::

Turn **Send email** off when another inject already tells the players where to read, such as a phone call simulated by an email from the communications team. The articles are still published at the inject's time.

::screenshot{id=openaev-crisis-injects-list alt="The Injects tab of Velvet Harbor crisis night: the email Call the crisis cell to the bridge at 40 minutes, the challenge Which account was used? at 50 minutes and Hotel Business Daily publishes the forum listing at 1 hour 30."}

## Check who read it

When a player opens the channel's page from their link, OpenAEV meets their reading expectation, with its full score. A player who never opens it fails the expectation when it expires: after the **Expiration time** of the expectation, 24 hours by default. Your administrator changes that default with `openaev.expectation.article.expiration-time`, or `openaev.expectation.human.expiration-time` for every expectation of people: see the [configuration reference](/reference/openaev/configuration/#openaev-expectation). Read the results on the inject, as for any expectation: see [Expectations](/openaev/use/expectations/).

The links in the emails are personal: a player who forwards one lets someone else read as them.

:::example[The press article of the Grand Meridian crisis night]
Example CERT runs the crisis exercise of Grand Meridian Hotels on the night of 9 January 2026, from 22:00. At 23:30, while the crisis cell is still assessing the intrusion, a trade newspaper says the guest records are for sale. In the scenario **Velvet Harbor crisis night**, the exercise planner writes the article:

| Field | Value |
| --- | --- |
| **Channel** | **Hotel Business Daily** |
| **Title** | `Guest records of Grand Meridian Hotels offered on a criminal forum` |
| **Author** | `Investigations desk` |
| **Content** | A seller on a criminal forum offers what he presents as the guest records of a European hotel chain, Grand Meridian Hotels: names, stay dates and partial card numbers. The chain has not answered our questions yet. |
| **Comments**, **Shares**, **Likes** | 48, 312, 95 |
| **Documents** | `forum-listing-grand-meridian.png` |

Then the inject `Hotel Business Daily publishes the forum listing`, of type **Publish a media pressure**, scheduled 1 hour 30 minutes after the start, targets the team **Crisis cell** with the subject `Hotel Business Daily mentions Grand Meridian Hotels`. At 23:42, Hélène Marchetti opens the article from her email: her expectation turns successful. Marc Dupuis, who never opens it, fails it the next night. The debrief starts from that gap.
:::

## What's next

- [Give your players challenges to solve](/openaev/use/components/challenges/).
- [Collect lessons learned after an exercise](/openaev/use/components/lessons/).
- [Send each player their own email with variables](/openaev/use/components/variables/), to name the player in the email.
