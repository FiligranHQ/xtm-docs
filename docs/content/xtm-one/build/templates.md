---
title: Brand the decks and reports agents produce
description: Give the decks and reports your agents generate your brand, with a template of palettes, typefaces, logos and assets that every deck wearing it follows.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/resources/templates/
---

A template is the design the presentations and reports of your agents wear: a palette for each theme, the heading and body typefaces, the logos drawn on every slide, the assets a deck may place (diagrams, screenshots, partner logos), the layouts it may use, and a few sentences of guidance on how to compose in that brand. Agents find templates by name, kind, aspect ratio, tags and supported themes, so the same request gives everyone on-brand output. This page creates a template, gives it its assets, and explains how decks use it; generating decks is in [Presentations](/xtm-one/use/presentations/).

::diagram{id=xtm-one-build-templates-linked}

## Before you start

- Templates are an AI area: the platform runs the Enterprise Edition and AI is switched on. Otherwise **Add Template** opens the Enterprise Edition notice.
- Any user can create a private template. Only an administrator can make it **Company-managed template** (shared with all users, editable by administrators only), and choose the platform's **Default Presentation Template** in :menu[Administration > Settings > Platform].

## How decks use a template

- **A deck stays linked to its template.** The palette, fonts and logos are read from the template every time the deck is drawn, so editing the template restyles every deck wearing it at once. The slides, their text and the images uploaded to the deck belong to the deck: deleting a template leaves its decks intact, in the plain look.
- **You choose it.** **New presentation** asks which template a deck starts in and which of its themes to use. In a chat, when you can see several templates, an agent asked for a presentation without a template named lists yours and asks which one before creating anything; name it in the request and it does not ask.
- **Automations use the default.** An assignment, or a chat where the agent cannot ask, uses the platform's default presentation template: the built-in **XTM One Brand Deck**, unless an administrator chose another company template.
- **Only the design travels.** A template restyles a deck through its palette, fonts, logos and footer pattern; it can never inject styling or scripts into the pages that render it.

The **XTM One Brand Deck** ships with every platform: the product's purple palette on dark surfaces, the XTM One logo, and the logos, diagrams and screenshots of the XTM suite as assets. Built-in templates carry the **Built-in** badge and are read-only, because every upgrade rewrites them. To start from it, open it and click **Duplicate** (under **More actions** when the window is narrower than 1280 pixels, with **Export** and **Delete template**): the copy is yours.

## Create a template

:::steps
1. Open :menu[Agent Studio > Templates] and click **Add Template**. The **New Template** page opens.
2. Fill **Name**, **Tags** and **Description (optional)**. Agents search templates by these, never by semantic search: say what the template is for.
3. Choose the **Kind**: **Presentation** for slide decks and pages, **Report** for executive reports. Agents look for a template of the kind they are producing.
4. Fill **Aspect ratio (optional)**, for example `16:9`.
5. For a presentation template, choose the **Themes this template supports**: **Dark**, **Light** or both (both by default, at least one stays on). A deck wearing the template can only use a supported theme.
6. To draft the design instead of typing colors, click **Generate with AI**: write a **Design brief** in plain words, add **Palette hints** (a hex value or a color word each), click **Generate**, judge the live preview, refine with **Refine this result** or start over with **Regenerate from the brief**, then **Apply to form**. Nothing is saved yet.
7. As an administrator, turn on **Company-managed template** to offer it to everyone.
8. Click **Create Template**. The template's page opens.
:::

::screenshot{id=xtm-one-build-templates-design alt="The Overview of the Grand Meridian briefing template: Presentation, 16:9, Light and Company-managed, its dark palette marked Not supported and its navy and brass light palette Supported, and the kind, theme and aspect ratio agents search by."}

## Give it its design and assets

A template's page has three tabs:

| Tab | What it is for |
| --- | --- |
| **Overview** | The description, the palette of each theme marked **Supported** or **Not supported**, the attributes agents search by, the layouts, and how often agents retrieved the template. |
| **Configuration** | The design editor: one palette per supported theme (the first one also carries the fonts), the chart series colors, **Deck defaults** (theme, aspect ratio, transition, title and closing covers), **Header & footer**, **Brand watermark**, **Corner emblem**, **Agent guidance**, the layouts and the sharing. **Preview the unsaved design** renders a sample deck from what you are editing. The raw settings stay under **Advanced (JSON)**. |
| **Assets** | The files the template brings to a deck. |

On **Configuration**, each color field shows its name above its key (**Background colour**, **Accent colour**, **Series 1**). **Success text**, **Warning text** and **Error text** are darker versions of the three tones, for the places a deck writes them as text (a check mark, a rising figure) when the tone itself is too light to read; left empty, they take the tone. Below **Corner emblem**:

- **Right footer pattern (optional)** replaces the slide number in the footer's right corner, for example `{{page}} · © {{year}}`. `{{page}}` is the slide number on two digits, counted from the first slide after the cover, and `{{year}}` the year the deck is drawn or exported.
- **Emblem size** (**Default size**, or 24 to 96 px) and **Emblem placement** (**In the top margin** or **Level with the slide title**) place the corner emblem.
- **Ink the emblem with the slide accent** draws the mark in the slide's accent color instead of its own colors, so a one-color mark reads on the dark and the light theme.

A template whose **Extra metadata** (under **Advanced (JSON)**) declares the brand cover, `"design": {"cover": "brand"}`, also shows **Cover emblem**, above **Header & footer**: the template's mark drawn faint behind the cover and the closing slide. **Size, % of the slide height** (60 to 140 %), **Cover emblem placement** (**Cropped by the bottom-right corner** or **Inside the right edge**) and **Cover emblem opacity** (3 to 10 %) set it. Left on **Default size** and **Default**, the mark is drawn at 95 % of the slide height, cropped by the bottom-right corner, at 7 %. It is sized on the slide height, so a 16:9 and a 4:3 deck draw it alike.

On **Assets**, click **Upload Assets** and give each file a role and a description:

- The role tells the renderer and the agents what the file is: `logo` (drawn on every slide), `logo_light` (the logo for the light theme), `logo_mark` (the symbol alone, used as watermark and emblem), and `diagram`, `screenshot`, `partner_logo`, `customer_logo` and the other roles of the list for what an agent places on request.
- The description is how an agent picks a file: write what it shows and whether it is drawn for a dark or a light surface.

Replacing a file keeps its identity, so decks that placed it show the new version. SVG files are cleaned of scripts and remote references on upload. **Export** downloads the template with its assets, and **Import** on the Templates page recreates it on another platform.

To build a template around a logo, create it, upload the logo as `logo` on **Assets**, then run **Generate with AI** from **Configuration** with the logo under **Sample colours from a logo**: the palette is sampled from it.

:::example[A Grand Meridian template for the crisis cell's briefings]
The **Crisis briefer** agent drafts the crisis cell's briefing as a deck. Hélène Marchetti, CISO and XTM One administrator, wants it in the Grand Meridian colors rather than the XTM One look. A company-managed template is created by an administrator, so she creates it herself:

| Field | Value |
| --- | --- |
| **Name** | `Grand Meridian briefing` |
| **Tags** | `crisis`, `executive` |
| **Description (optional)** | `Briefings for the crisis cell and the executive committee of Grand Meridian Hotels.` |
| **Kind** | **Presentation** |
| **Aspect ratio (optional)** | `16:9` |
| **Themes this template supports** | **Light** only: the briefings are printed |
| **Generate with AI**, **Design brief** | `A sober light deck for a hotel group's crisis briefings, deep navy and brass, understated, readable when printed.` |
| **Company-managed template** | On |

On **Assets**, she uploads the Grand Meridian logo as `logo_light` and the hotel network map as `diagram`, described `Map of the Grand Meridian hotels in France and Spain, drawn for a light surface`. On **Configuration**, she sets **Right footer pattern (optional)** to `{{page}} · © {{year}} Grand Meridian Hotels`, so every printed page carries its number and the copyright. She then makes it the **Default Presentation Template** in :menu[Administration > Settings > Platform]. The next briefing on Velvet Harbor that **Crisis briefer** generates opens on the Grand Meridian logo and places the hotel map when it lists the exposed hotels.
:::

:::xtm-one[Let an agent design the template]
The built-in **Template Designer** agent creates and refines a template from a conversation: describe the brand, answer its questions, and it writes the palette, the layouts and the guidance, attaches the logos you uploaded to the chat, and renders a preview deck to judge. It cannot read colors out of an image: for that, use **Generate with AI** with **Sample colours from a logo**, then refine with the agent.
:::

## Check that it works

On the template's page, click **Preview**, or **New presentation with this template**: the sample deck shows your palette, fonts and logo. In a chat, ask an agent with **Presentation Generation** for a deck "in the Grand Meridian briefing template": it uses it without asking.

## What's next

- [Presentations](/xtm-one/use/presentations/), to generate, edit and present decks.
- [Share agents and resources](/xtm-one/build/sharing/), to offer a template to a group.
- [Write custom tools](/xtm-one/build/tools/), for the built-in tools that generate files, reports and decks.
