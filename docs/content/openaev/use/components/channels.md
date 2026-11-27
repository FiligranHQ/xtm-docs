---
title: Create the media outlets of an exercise
description: Create the newspaper, social network or TV channel your players read during a crisis exercise, with its name, colors and logos, before you write its articles.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/components/channels/
---

A channel is a fake media outlet: a newspaper, a social network or a TV channel, with its name, colors and logo. During a crisis exercise, players open the articles you publish on the page of their channel, which looks like the real thing. Create the channel once; every scenario of the platform can publish articles on it. This page creates a channel and gives it its look. Writing and publishing the articles is in [Publish fake press articles during an exercise](/openaev/use/components/media-pressure/).

## Before you start

- Your role has **Manage channels** to create and change a channel, and **Delete channels** to delete it.
- To set a logo, your role also has **Manage documents**: a logo is an image of :menu[Components > Documents].

## What a channel decides

::diagram{id=openaev-channels-article-display}

The channel gives the page its frame, the article gives the content. The **Type** decides both the layout and what an article can carry:

| Type | The players see | An article can show |
| --- | --- | --- |
| **Newspaper** | A front page: the latest article as the headline, the others below, each with **Read more** | Images |
| **Microblogging** | A timeline of posts, with their comments, shares and likes | Images and videos |
| **TV channel** | Videos with their titles | Videos |

The newest article comes first. Each player sees the logo and colors of the theme they use, dark or light.

## Create a channel

:::steps

1. Open :menu[Components > Channels] and click **Create**. The **Create a new channel** panel opens.
2. Choose the **Type**: **Newspaper**, **Microblogging** or **TV channel**.
3. Fill in the **Name**, as the outlet signs its pages, and the **Subtitle**, the line under it. Both are required.
4. Click **Create**.

:::

The channel appears in the list with its **Type**, **Name** and **Subtitle**.

## Give it its look

:::steps

1. Click the channel in the list. Its page opens with **Parameters**, **Branding** and a **Live preview**.
2. Under **Parameters**, choose the **Header mode**: **Title only**, **Logo only**, or **Logo and title**. A new channel shows the title only.
3. Pick the four colors, or type their hexadecimal codes, such as `#0b3d91`. **Primary color (dark)** and **Secondary color (dark)** serve players in the dark theme; **Primary color (light)** and **Secondary color (light)**, the light theme.
4. Click **Update**.
5. Under **Branding**, click **Change logo** on the **Dark theme** tile, choose an image among the documents or upload one, then do the same on the **Light theme** tile. A logo is a PNG, JPEG, SVG or GIF file.

:::

The **Live preview** shows your edits before you save them; its two buttons switch it between the dark and the light theme.

Changes apply at once to every article of the channel, in every scenario and simulation, published or not.

::screenshot{id=openaev-channels-channel alt="The channel Hotel Business Daily, a newspaper: its type, header mode, name, subtitle and the primary and secondary colors of each theme, with the live preview of its page and the Branding section for its logos."}

## Delete a channel

**Delete**, in the menu of the channel's line, deletes the channel and every article written for it, in every scenario and simulation. To retire a channel that past exercises used, keep it and stop publishing on it.

:::example[The newspaper of the Grand Meridian crisis night]
For the crisis exercise of 9 January 2026, Example CERT wants the crisis cell of Grand Meridian Hotels to read a trade newspaper break the news of the guest records theft. The exercise planner creates the channel:

| Field | Value |
| --- | --- |
| **Type** | **Newspaper** |
| **Name** | `Hotel Business Daily` |
| **Subtitle** | `News of the hospitality industry (a fictional newspaper of the exercise).` |
| **Header mode** | **Title only** |
| **Primary color (dark)** | `#e3b341` |
| **Primary color (light)** | `#7a1f1f` |
| **Secondary color (dark)** | `#2b2b2b` |
| **Secondary color (light)** | `#f4efe6` |

The articles of the scenario **Velvet Harbor crisis night** then open as the front page of Hotel Business Daily. Hélène Marchetti, who reads in the light theme, sees its masthead in burgundy.
:::

## What's next

- [Publish fake press articles during an exercise](/openaev/use/components/media-pressure/).
- [Share files with your players](/openaev/use/components/documents/), for logos and article images.
