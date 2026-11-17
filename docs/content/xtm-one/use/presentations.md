---
title: Turn an agent's work into a presentation
description: Have an XTM One assistant build a slide deck or a one-page brief from a conversation, change it by asking or by hand, present it with your notes, share it and export it to PDF or PowerPoint.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, executive]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/presentations/
---

An AI assistant of XTM One builds a slide deck for you from a request in plain words: the briefing of the crisis cell, the quarterly threat review, a one-page brief for the executive committee. The deck appears in the conversation, plays like a real deck with your speaker notes, stays editable by asking for changes or by hand, and exports to PDF and PowerPoint that look like the slides on screen. Every deck you can see is listed in :menu[Work > Presentations]. How a deck gets its brand (fonts, colors, logos) is in [Start from templates](/xtm-one/build/templates/).

::diagram{id=xtm-one-presentations-lifecycle}

## Before you start

- Presentations need the Enterprise Edition and AI switched on.
- The assistant you ask carries the **Presentation Generation** tool. On XTM One, the **CTEM Assistant** (the General Assistant), the **XTM-One Assistant** and the assistant of each product (**OpenCTI Assistant**, **OpenAEV Assistant**, **OpenCRQ Assistant**) carry it from the start.

## Create a deck

:::steps
1. In a conversation, ask for slides in your own words. Name the audience, the one message to remember and the numbers you want on screen.
2. When you can use several templates and your request names none, the assistant lists them and asks which one to use. Answer with a template's name, or "no template" for the plain look.
3. Read the deck in the card that appears in the conversation. Ask for changes in the same conversation: see [Change a deck](#change-a-deck).

::screenshot{id=xtm-one-presentations-chat-card alt="Crisis briefer's answer to Hélène Marchetti with the card of the Velvet Harbor crisis briefing in the conversation: the deck in the Grand Meridian briefing template on its title slide, and its toolbar with the slide counter 1 / 8, Present, comments, sharing, download and Open in studio."}
:::

You can also start from :menu[Work > Presentations]: **New presentation** asks for the template (one you can see, **No template - plain look**, or **Let my assistant ask me in the chat**), the theme, and the assistant that writes it (**Written by**), then opens the chat with the start of the request typed for you. Automations, and chats where the assistant cannot ask, use the platform's default template, set by an administrator.

A deck can hold text, lists, key figures, tables, timelines, process flows, quotes, images from the template's assets, videos, native charts, org charts, world maps by country, positioning maps, diagrams drawn from a description, formulas ("A + B = the offer"), configuration cards and [live polls](#ask-the-room-with-a-live-poll). Ask for a **one-pager**, a **product brief** or a **datasheet** and the assistant lays it out on a portrait A4 or US Letter page, read rather than projected. For a deck you make every month, ask the assistant to copy the last edition ("copy *Velvet Harbor review, March 2026* as *Velvet Harbor review, April 2026*, then update the numbers"): the copy keeps the template, the theme and every slide with its notes.

### Place options on a positioning map

A **Positioning map** places up to 30 items on two axes, such as the hotels of a chain by their exposure and their readiness. Each axis says what it measures and what its two ends mean, and the four quadrants carry a name over a faint tint. The item the slide is about stands out, larger and in the template's accent; the others can be dimmed as context. Up to four callouts, drawn with the map, say why that item stands where it does, and each item can carry a one-sentence note that shows when you point at it. XTM One puts each name on a free side of its mark, so that names do not cover one another; on a crowded map, **Layout check** reports **Map labels overlap**.

Ask the assistant for one, naming the axes, the items and where each stands ("a positioning map of our three hotels by exposure and readiness, Madrid highlighted"). In **Edit slides**, **Selection** changes its **Axes** (**Crossing at the centre**, a two-by-two, or **Along the edges**) and puts its **Callouts** **Under the map** or **Beside the map**.

::screenshot{id=xtm-one-presentations-positioning-map alt="A slide of the Grand Meridian briefing template with a positioning map of the Paris, Madrid and Barcelona hotels by exposure and readiness: four named quadrants, Madrid highlighted in the Act now quadrant, and a callout under the map saying why."}

### Play a video on a slide

A **Video** plays inside the slide. Give the assistant the video's link:

- **A YouTube or Vimeo link** plays in that site's own player, which loads only while its slide is on screen.
- **A link to an MP4 or WebM file** on a public `https://` address: XTM One downloads it once, when the deck saves, and keeps it as a file of yours, so the slide still plays when the link stops working. A file over 100 MB, or a link that serves no MP4 or WebM video, is refused and the deck is not saved.

A video loads only when its slide comes on screen, and stops when the slide leaves, so a deck of several videos opens as fast as any other. In **Edit slides**, **Selection** sets how it plays: **Autoplay** (it starts when its slide shows, muted unless you turn **Muted** off), **Loop**, **Controls** (the play button, the volume and the timeline, on by default), and **Start at** and **End at**, in seconds, to play one passage only (a Vimeo video takes the start only). **Frame** draws a browser window or a device around it, as for an image.

Positioning maps and videos are not in **Add a block**: ask the assistant to place them, then adjust them by hand.

## Present it

The card in the chat is a live deck, with its toolbar along the top edge:

- Previous and next, and the slide counter (`3 / 12`), which opens a list of every slide by title.
- **Present** takes the deck to the full screen. `F` presents, `Esc` stops. A click on the slide shows the next one, Shift + click the previous one.
- **Open in studio** opens the deck on its own page, `/presentations/` followed by its identifier, with the editor, the comments, the history and the sharing: it is also the page to put on a projector.
- **More actions** holds the slide overview (`O`), the black screen (`B`), the **Speaker view** (`S`), **Present to a meeting** (`A`), the **Notes handout** and the keyboard shortcuts (`?`).

::screenshot{id=xtm-one-presentations-deck-card alt="The Velvet Harbor crisis briefing opened in the studio, on its own page: the title slide by Hélène Marchetti in the Grand Meridian briefing template, the slide counter 1 / 8, Present, and the comments, history, editing and sharing of the toolbar."}

**Speaker view** opens in a window of its own: the current slide, your notes in large type, the next slide, a stopwatch and your pacing against the speaking time of each slide's notes. It follows the deck on the room's screen, and your clicker moves both. Your browser may block the window: allow pop-ups for XTM One. **Notes handout** opens every slide with its notes as one printable page.

A portrait page opens as a document instead: pages stacked on a scrolling surface, with zoom controls and **Fit width**, and no speaker view.

### Present to a meeting

When the audience is on a Teams, Zoom or Google Meet call, share a window that shows the slides and nothing else.

:::steps
1. On the card or the deck's page, open **More actions** and pick **Present to a meeting**, or press `A`. Two windows open: the audience window and the **Speaker view**.
2. In the meeting tool, share the audience window: it carries the deck's title. It shows the slides on black, with no toolbar and no pointer.
3. Present from the **Speaker view**: the arrow keys and your clicker move the slides. A click on the audience window shows the next slide, Shift + click the previous one.

::screenshot{id=xtm-one-presentations-speaker-view alt="The speaker view of the Velvet Harbor crisis briefing before the meeting: Rehearsal - no presentation window connected and No audience window in its header, Open audience window and Hand the clicker, the title slide with Hélène Marchetti's notes, the agenda as the next slide, and the stopwatch."}
:::

The header of the **Speaker view** says whether a window shows the deck (**Following the presentation**) and whether the meeting sees the slides (**Audience window live** or **No audience window**). **Open audience window** opens the window to share again, at the current slide; once it is open, **Bring back audience window** brings it back. With two screens, in a browser that manages windows (not Firefox or Safari) and that you allow to, it opens full size on the other screen. In the audience window, `S` brings back the speaker view and `Esc` does nothing: close it with its own close button. When your browser blocks the window, XTM One says **Audience window blocked**: allow pop-ups for XTM One.

**Hand the clicker**, in the header of the speaker view, lets a colleague move your slides from their own computer. Pick them in **Person to hand the clicker to**: only people who can already open the deck can take it, so share it with them first. They are told, and the deck's page shows them **You hold the clicker**, with previous and next, the slide number and **Give back**. You see who holds it and can **Take back** at any time; your own keys keep working.

### Ask the room with a live poll

A **Live poll** puts questions on a slide that the audience answers from their phones, without an account. Ask the assistant for one ("add a quiz on the closing slide"), or, in **Edit slides**, pick it in **Add a block**.

- A poll holds up to five **Questions**, each of a kind: **Choice** or **Quiz** (two to six options, a quiz with one right answer), **Word cloud** (one to three words per person, rude words refused) or **Rating 1-5**.
- The slide shows a short link and, with **Show the QR code**, its QR code. People vote anonymously, once per question from each phone, 500 at most per poll. The results on the slide refresh every two seconds.
- **Results** shows the votes in **Real time** or **After lock**. **Mode** is **Guided**, where you open the questions one at a time, or **Autonomous**, every question open at once with nothing to drive. **Quiz answer** says when the right answer shows: **Manual**, **From the start**, **After votes** (10 unless you set another number), **After a delay** (20 seconds after the first vote unless you set another) or **Never revealed**. **Presenter controls** puts a bar on the presented slide (**On hover**) or nothing (**No controls**).

The owner and the editors drive the poll under the deck and under the current slide of the speaker view: **Previous question** and **Next question**, **Reveal the answer**, **Lock voting** and **Reopen voting**, and **Reset votes**, confirmed with **Delete all votes**. On the full screen, `N` and `P` step through the questions, `V` reveals the answer, `L` locks voting and `Shift+R` resets the votes. A public link shows the questions only; an export draws the link, the QR code and the results as they stand when you export. What the room sees on their phones is in [Let your audience vote from their phones](/xtm-one/use/live-poll-voting/).

:::note
The audience votes through a public page: when an administrator turns public links off (see [Show it to people without an account](#show-it-to-people-without-an-account)), nobody can vote.
:::

## Change a deck

Ask the assistant in the same conversation, and it changes the deck in place: the card and the link you shared show the new version. A small change stays small: "change *revenue* to *loss* on slide 4" edits that word on that slide, and nothing else is redrawn. Name the slide ("on slide 4", "the closing slide") to keep the change where you want it. Ask "which slides talk about the Madrid hotel?" to find a word without changing anything.

- **Edit slides**, on the deck's page, turns the deck into an editor: click a text and type, select a block to move, duplicate or delete it, and use the side panel. **Slides** reorders, duplicates, deletes and adds slides; **Selection** holds every setting of the slide or block you selected; **Deck** holds the title, the template, the theme (**Dark** or **Light**, among those the template supports) and the header and footer. Every change saves itself.
- **Studio** opens a chat beside the deck: the assistant knows which deck and which slide is on your screen, so "shorten this title" is enough. The agent picker offers the agents that carry **Presentation Generation**; **Continue with another agent** hands the conversation to another of them, its history kept. **Conversations about this presentation** lists the deck's earlier conversations, and **Open in chat** takes the current one to the chat page. The deck refreshes as it saves, and its reply links the slides it changed.

Several people can edit a deck at the same time: their changes merge with yours, unless you both changed the same block.

## Share it, review it together

**Share** (on the card, on the deck's page or on the library card) opens the sharing dialog. A deck starts private to you.

- Add people or groups with **Can view** or **Can edit**, then copy the link. **Who can open this link** says who it opens for.
- **Can view** opens, presents, exports, duplicates, comments and browses the history. **Can edit** also edits, by hand or with their own assistant in the studio. **Copy and edit with my assistant** gives a viewer a copy of their own.
- Administrators can publish a deck to the whole company.

**Comments** opens the review notes: a note is attached to the slide you are on, and clicking it goes back to that slide. Ask your assistant to "address the review comments", and it changes the deck and resolves the notes. **History** keeps every change as a numbered revision, with who made it and what changed, the last thirty per deck; you can preview one and restore it.

### Show it to people without an account

Turn on **Public link** at the bottom of the sharing dialog to give the deck a second address that anyone can open without signing in: the slides only, read-only, never the notes, comments or history. **Allow PDF download** adds a **Download PDF** button to that page. **Replace link** issues a new address and stops the old one at once; turning the switch off stops it for everyone.

:::warning
Anyone who has a public link can open the deck and pass the link on. Turn it off, or replace it, when it is no longer needed. Administrators can turn public links off for the whole platform in :menu[Administration > Settings > Access and security], under **Security options** (**Public presentation links**), or with `PRESENTATION_PUBLIC_LINKS_ENABLED=false`.
:::

## Export it

**Export**, on the card and on the deck's page:

| Format | What you get | Use it when |
| --- | --- | --- |
| **PDF (vector)** | One page per slide, with selectable text, the deck's fonts and sharp charts | You send or archive the deck |
| **PowerPoint (pixel-perfect)** | One image per slide, with the speaker notes and the transition | The deck is final and is presented from PowerPoint |
| **PowerPoint (editable shapes)** | Text boxes, tables and native PowerPoint charts placed as on screen | Someone keeps working on it in PowerPoint |
| **JSON package (import elsewhere)** | The deck itself, with its images, in one file | You move the deck to another XTM One platform: **Import** on the Presentations page reads it back |

A video cannot play on paper: the PDF shows a still with a play badge, linked to the video's page for a YouTube or Vimeo video, and **PowerPoint (editable shapes)** turns a video file of up to 100 MB into a movie that plays in PowerPoint.

**Print to PDF (browser)**, in **More actions**, prints from your browser when the platform's export is not available; pick **Save as PDF** as the destination.

## Read and export a report

A deck is made to be presented, slide by slide. A report is made to be read: one document, top to bottom, that prints as pages. Ask an assistant for "a report", "an executive summary" or "an assessment", and it writes one with its **Report Generation** tool, which the assistants that build decks carry from the start.

A report opens on a masthead: its title, tags such as the date, the classification and the audience, and the bottom line, graded. Its numbered sections, which also build its table of contents, are chosen among fixed kinds: key figures, text, decisions with their confidence, an attack chain, findings with the evidence and the confidence behind them, scenarios, a risk table or a two-way risk matrix, a timeline, a checklist, a form to fill in, annexes that fold, and the sources it cites. The assistant fills the sections; the layout and the print style are XTM One's, the same for every report.

- **In the conversation**, the report appears as a card with its title and number of sections, scrolling inside the chat. **Read fullscreen** enlarges it; **Open in a new tab** opens the report viewer.
- **The report viewer**, `/reports/` followed by the report's identifier, shows the report alone, full width, without the menus of XTM One, with its title as the tab's name.
- **To change it**, ask in the same conversation ("lower the verdict to *moderate* and add the Madrid hotel to the timeline"): the assistant rewrites the report in place, and the card and the viewer show the new version.

**Export**, on the card and at the top right of the viewer, offers two formats:

| Format | What you get |
| --- | --- |
| **PDF (vector)** | The platform renders the report as a PDF with selectable text and downloads it |
| **Print to PDF (browser)** | The viewer opens in a new tab with every annex unfolded and your browser's print dialog: pick **Save as PDF** |

A report has no sharing: only the person whose conversation produced it, and the administrators, can open it; anyone else reads **This report is no longer available.** To give it to others, export the PDF, or ask the assistant to save the PDF as a file of the conversation. In a messaging app, the answer carries a link to the viewer, which opens for the same people.

## Find your decks

:menu[Work > Presentations] lists every deck you can see, the most recently changed first, and how many it lists beside the title. Search by title, owner or template, filter by **Kind** (**Decks** or **Pages**), and narrow to **Mine**, **Shared** or **Company**: each shows how many decks it holds, and the page opens on the first one that holds any.

The decks show as cards or, with **List view** at the right end of the toolbar, as a table that compares them at a glance:

- **Name**: the title, and who made the deck when it is not you.
- **Size**: the number of slides, or of pages for a portrait page.
- **Template**: the template, the theme and, for a page, its paper size.
- **Comments**: the number of open review notes. On a card, the number opens the deck on its comments; in the table, click the row to open the deck.
- **Sharing**: whether the deck is shared, published to the company, view-only for you or open through a public link.
- **Updated**, and **Created**, which **Columns** brings in.

Click **Name**, **Size**, **Updated** or **Created** to sort, again to reverse; on the cards, pick the column in **Sort by**. Both views share the order, the search and the pager, which shows 10, 25 (the default), 50 or 100 decks. XTM One remembers the view you chose.

::screenshot{id=xtm-one-presentations-library alt="The Presentations library as a table: the Velvet Harbor crisis briefing and the crisis cell's vote deck with their size, template, comments, sharing and last update, under the search, the Kind filter and the Mine, Shared and Company segments."}

**Templates**, at the top, opens the templates the decks are dressed with. On a card or a row, **Present** (**Open** for a page) shows the deck, **Edit with my assistant** opens the studio, and **Presentation actions** holds the other actions. Deleting a deck removes it for everyone it was shared with.

:::example[Brief the crisis cell on Velvet Harbor]
On 12 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels, needs a deck for the crisis cell's meeting the next morning. She opens a conversation with **Crisis briefer**, which carries **Presentation Generation**, and writes:

> Create an 8-slide briefing for the crisis cell on the Velvet Harbor intrusion of 9 January 2026: what happened at the front desks, the guest records taken, what we changed since, and the three decisions we need. Speaker notes on every slide, two sentences each.

The assistant asks which template to use; she answers `Grand Meridian`. The deck appears in the conversation. She asks "on slide 5, replace *Spain* with *Madrid and Barcelona*", then opens **Share** and gives the group `Crisis cell` **Can view**. She also asks for a live poll on the decisions slide: "which decision comes first?", a **Choice** question with the three decisions as options.

The next morning, the Madrid managers join over Teams. She picks **Present to a meeting**, shares the window named *Velvet Harbor crisis briefing* and talks from the **Speaker view** on her laptop. On the poll slide, the cell votes from their phones; she locks voting and reads the result aloud. She then sends the **PDF (vector)** export to the executive committee.

For the committee's next meeting, she asks for "a positioning map of our three hotels by exposure to Velvet Harbor and readiness, Madrid highlighted, and say why it stands there". The slide *Madrid is the hotel to act on* places Paris in the quadrant *Hardened*, Barcelona and Madrid in *Act now*, Madrid larger, with the callout *No detection in Spain yet* under the map.
:::

## What's next

- [Start from templates](/xtm-one/build/templates/), to give every deck your brand.
- [Find the files agents created and received](/xtm-one/use/files/): each export is a file there.
- [Get answers from an agent in the chat](/xtm-one/use/chat/).
