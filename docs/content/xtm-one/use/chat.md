---
title: Get answers from an agent in the chat
description: Ask XTM One AI agents questions in plain language, bring several agents and earlier conversations into one message, get files back and continue the conversation in Slack or Teams.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, soc-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-16
edition: ee
migrated_from:
  - xtm-one:/usage/chat/
---

The chat is where you talk to XTM One AI agents in plain language: ask for the context of an alert, a summary of a report, a translation, a draft. You choose who answers, the General Assistant or a specialized agent, you attach the documents the answer depends on, and the agent works with its tools and knowledge until it answers. You can bring other agents and earlier conversations into the same message, ask for changes to a file the agent produced, and carry on in Slack or Microsoft Teams. What an agent is and how it is built is in [What agents are made of](/xtm-one/concepts/agents/).

::diagram{id=xtm-one-chat-turn}

## Before you start

- The chat needs the Enterprise Edition and AI switched on. On the Community Edition, or when an administrator has switched AI off, :menu[Chat] stays in the menu with a lock: see [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/).
- The agent you want is shared with you and open to chat. An agent built for automations still shows, marked as not recommended for chat.
- Each answer counts as one agentic execution against your quota. Your profile's **Quotas** tab shows what is left: see [Set up your profile and API keys](/xtm-one/use/profile/).

## Start a conversation

:::steps
1. Open :menu[Chat] in the left navigation. Your conversations are listed on the left; the open one fills the rest of the page.
2. Click **New conversation**.
3. At the top of the conversation, open the agent switcher and choose who answers. Type in it to find an agent by name:
   - **General Assistant**: a broad helper with the platform's default tools. When an administrator links an agent to it, that agent takes its place, marked **Assistant**.
   - An agent: a focused role with its own instructions, knowledge and tools, such as **Night shift triage**. **Company** marks the agents your company manages.
   - An agent under **Not recommended for chat**: built for assignments. It answers, but it was not designed for a conversation.
4. Type your message and send it. The answer appears as the agent writes it.

   ::screenshot{id=xtm-one-chat-conversation alt="Paul Garnier's conversation with Night shift triage: the EDR alert on FD-PAR-03 at 02:04, and the answer To escalate, with the evidence from OpenCTI, the same path as the January incident and the next step; his conversations are listed on the left, the composer waits at the bottom."}
:::

A new conversation starts with the default agent of your profile. Without one, it starts with the agent marked **Assistant**. On XTM One, without either, the **CTEM Assistant** answers and routes your request to the right specialist. Opening the chat from an agent's card or page selects that agent for you. The agent's name, at the top of the conversation, opens its page.

A conversation keeps its context and its agent: open it again from the list to go on where you stopped. To have another agent answer it, [hand it over](#hand-the-conversation-to-another-agent).

:menu[Chat] opens on the conversation you opened last, or on your most recent one, rather than on an empty page: click **New conversation** to start another. When you archive the conversation you have open, the page shows a new, empty conversation; the next time you open :menu[Chat], it opens your most recent remaining one. On a phone, or in a window narrower than 1,024 pixels, the list of conversations starts closed.

## Write your message

The composer at the bottom takes more than text:

| To | Do |
| --- | --- |
| Give the agent a document | Click the paperclip, paste the file, or drop it on the page. A message takes up to 30 files, of up to 40 MB each |
| Start from a saved request | Click the prompt button and pick a prompt: its text goes into the composer. Prompts are built in [Write reusable prompts](/xtm-one/build/prompts/) |
| Speak instead of typing | The microphone turns your speech into text in the composer, when your browser supports it |
| Talk with the agent | **Start voice conversation**, the phone button, starts a call where the agent answers aloud. It shows only when your browser supports calls |
| Add something while the agent works | Write and send: your message joins the answer under way, or leaves as soon as that answer ends |
| Interrupt an answer | While the agent writes, **Stop** ends the answer |

A call answers as fast as the agent's model: a tip in the call suggests a quick model, such as GPT-4o mini or Gemini Flash, for faster spoken answers. Close the tip to hide it for the rest of your session.

The quota indicator among the composer buttons shows the agentic executions you used today, out of your limit when you have one. Click it to see today, this month and this year. An answer that fails may be credited back. The composer also shows how much of the model's context window the conversation fills.

A line under the composer reminds you that AI can make mistakes. Check what matters before you act on it.

## Ask several agents at once

Type `@` and pick an agent to mention it. Mention several to get each one's answer to the same message:

- In a new conversation, the first agent you mention becomes the conversation's agent. The conversation's agent answers first, then each mentioned agent in the order you named them, each in its own message labelled with its name.
- Every agent receives your whole message and its attachments, and reads the answers given before its own.
- A mention of an agent you cannot chat with (not shared with you, disabled, not open to chat) is ignored; the others still answer. An agent you mention twice answers once.
- Each answering agent counts as one agentic execution. When your quota runs out, the agent that could not be charged says so, and the agents after it are not asked.
- **Stop** ends the turn: the agents that have not answered yet are not asked.

In Slack, Teams and the other messaging channels, mentions work only once an administrator turns them on: see [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

## Point the agent at another conversation

To have the agent read what was said elsewhere ("what did we decide there?"), type `@`: under the agents, **Reference a conversation** lists the conversations you can open, the most recent first. Type words of the title to narrow it, then pick one. The composer inserts its key, the title in lowercase with hyphens, such as `@velvet-harbor-triage`.

- The agent receives a short brief of each referenced conversation (title, summary, last messages) and, for the rest of the conversation, can read it in full while you can still open it.
- The list holds your own conversations, including those of the assistant embedded in OpenCTI and OpenAEV, and those shared with you. It leaves out archived and empty conversations and the one you write in.
- Only a conversation picked from the list counts: an `@` word you type by hand stays plain text.
- A message references at most 5 conversations. Each reference shows as a chip on your message; a reader of your conversation opens only the ones they may open.

## Get files back, and ask for changes

Files an answer names show as cards under it; the other files the agent wrote while working (a first draft, a scratch file) are under **Working files**. When the agent wrote the same file twice in one answer, the older one is marked **Superseded**. Files of the same name in a conversation are numbered `v1`, `v2` in the order they were written. A link in the text of an answer to one of the conversation's files downloads it, as its card does. Click a picture or a chart of an answer, or an image you attached, to see it full size.

To change a file the agent produced, say what to change in plain words: "remove the paragraph about the Madrid hotel", "rename the column to Severity". The agent applies exactly that change and returns a new revision: the version you downloaded never changes under you. A text file (HTML, Markdown, CSV, JSON, YAML, code) is edited in place; a PDF or Word document is generated again from the corrected content. Quote the words as they appear in the document: the agent matches them against the file's exact text.

Every file of your conversations is also listed in :menu[Work > Files]: see [Find the files agents created and received](/xtm-one/use/files/).

### When a file was deleted

XTM One does not keep every file for good: an administrator sets how long each kind of file is kept. By default, the files an agent hands you and its working files go after 60 days without use; downloading a file, or an agent reading it, counts as a use. The files you attach are kept unless an administrator sets a limit. A file that an open work item, a presentation or a report still uses is kept.

When a file's content is deleted, its conversation keeps a card where the file was: its name and size, greyed, with when and why it went, such as `Deleted on Jun 12, 2026 by the retention policy`. A working file shows struck through. The card no longer downloads, and a link to the file in an answer shows **Download failed**, with `"indicators-to-block.csv" was deleted by the retention policy.` The agent reads the same notice instead of the file: ask it to produce the file again. **Conversation options** > **Conversation files** lists a deleted file with the same notice, without **Download** or **Add to the workspace's sources**. Download the files you want to keep. How long each kind of file is kept is set in [Settings](/xtm-one/administer/settings/).

## Rate an answer

Point at a finished answer to show its thumbs, **Helpful response** and **Not helpful**. A thumbs down opens a dialog to say what went wrong. When administrators have defined [feedback criteria](/xtm-one/administer/settings/), any rating opens it, to score each criterion. The people who manage the agent read every rating in its **Feedback** tab.

- Click the same thumb again, before you reload the page, to take your rating back.
- A rating you gave earlier shows on its thumb, with **Feedback already submitted**: it is final.
- A conversation shared with you read-only has no thumbs.
- An administrator also sees **View trace** under the answer, which opens its trace.

## Choose the tools of the conversation

The wrench button, **Session tools**, opens the menu of the tools this conversation may use:

- With the **General Assistant**, you build a toolset for this conversation: built-in tools, custom tools, integrations, MCP servers and knowledge bases.
- With an agent, the menu starts from the agent's own toolset, and you switch its built-in tools on or off for this conversation. Its integrations are the ones the agent is connected to.

The menu has one entry per kind of tool, **Built-in**, **Custom Tools**, **Integrations**, **MCP Servers** and **Knowledge Bases**, each with the number of its tools switched on. Point at a kind to list its tools, each with a switch: click a tool to switch it on or off. The menu stays open, so you can switch several in a row.

- An integration that serves several platforms, such as two OpenCTI instances, opens its own list: its switch, then under **Platform**, **All platforms** or the one to use.
- **Show or hide integrations**, at the bottom of **Integrations**, opens the list of your integrations: a hidden one stays out of the menu until you show it again.
- **Reset default**, **Enable all** and **Disable all**, at the bottom of the menu, act on the whole selection.

Your selection is saved with the conversation when you send a message, for the agent that answers it. Open the conversation again and the menu shows it as you left it. A tool you switched off stays off; a tool added to the agent since then is on. Its turns in Slack or Teams, and the replies the agent writes when background work ends, use the same selection. Once you [hand the conversation over](#hand-the-conversation-to-another-agent), the new agent starts from its own tools. What you pick before the first message of a new conversation stays with that conversation only.

You can only add what you could connect to an agent of your own: a tool, MCP server, knowledge base or integration you cannot see is refused, with the kind named, and nothing is sent. An integration whose owner's account is not shared with you is never used for you: a note under the answer names it and says why, so connect your own. Your profile keeps your own defaults for the General Assistant: see [Set up your profile and API keys](/xtm-one/use/profile/).

## Organize your conversations

The list shows your conversations by date, each with the icon of its agent. Drag its edge to widen it: your browser keeps the width, as it keeps the draft you leave in the composer. Search the list to find a conversation. Under your own, **Shared with me** lists the conversations others shared with you that match the search by title, owner or last message.

Archive a conversation from its row, or every one with **Clear all conversations**: an archived conversation leaves the active list.

To keep the conversations of one investigation together, file them in a workspace: drag a conversation onto a workspace in the list, or open **Conversation options** > **File in a workspace**. Filing into a workspace other people can see asks first, in **File into a shared workspace?**: its members can then search what was written in the conversation. Click **File it** to go on. Once a conversation has moved, a notice names the workspace it went to and lets you undo the move. The agents of a workspace's conversations share its work items, its shared state and its knowledge. See [Share a workspace with your team](/xtm-one/use/workspaces/).

## Hand the conversation to another agent

The agent you choose in the agent switcher answers new conversations: an open conversation keeps its own. To keep this one, with its whole history, and have another agent answer from now on, open **Conversation options** > **Continue with another agent**:

:::steps
1. Pick the agent: the list holds the agents you can chat with, other than the one answering now, and the **General Assistant** when an agent answers now. Type in **Search agents...** to narrow it.
2. Click **Continue**.

   ::screenshot{id=xtm-one-chat-continue-with-agent alt="The Continue with another agent dialog over Paul Garnier's conversation with Night shift triage: the conversation keeps its whole history, the list of agents to pick from, and the Continue button."}
:::

The agent you picked answers from your next message and reads the earlier answers as the previous agent's. A line across the transcript marks the change: which agent continued, and when.

- Only the owner hands a conversation over, and only a conversation held in XTM One: a Slack or Teams thread stays with the app's bot. The option is unavailable while an answer is being written.
- When an agent is deleted, its conversations are kept with their history. If whoever deleted it named a successor, the successor answers them, and the line says the agent was deleted and who continued. Otherwise, the dialog tells you the conversation's agent was deleted: pick the one to continue with.

## Continue in Slack or Teams

A conversation is not tied to where it started. Open it, then **Conversation options** > **Continue in Slack or Teams**, and pick the app: the bot sends you a direct message, and the conversation goes on there with its history, files and agent. A direct-message conversation you had with the Slack or Teams bot appears in your list, marked with the app's icon, and goes on here when you write in it.

- Each answer is written where you sent the message.
- Only the owner continues a conversation; one shared with you stays read-only. Slack channel threads and Teams group chats stay in their app.
- A tool call that waits for your approval is answered in XTM One: Slack and Teams cannot show the approval card.
- **Stop**, in the same dialog, ends the continuation.

The option appears only when the conversation's agent answers on that app and the bot takes direct messages. Bots are set up by an administrator: see [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

## Export a conversation

**Conversation options** > **Export as Markdown** or **Export as PDF** downloads the conversation: every message in order, with who wrote it and when, a chart written as a table of its values. Anyone who can read the conversation can export it; the 2,000 most recent messages are exported.

## When an answer takes long or stops

- **Background work**: when an agent delegates part of the request, a banner shows while the background tasks run, then a completion banner with the number that failed, if any. When a task ends, the agent takes the conversation up by itself and reports on it, with the files the task produced. That reply answers no new message of yours, and a line above it names the work it reports on: "Background task of Velvet Harbor watcher finished", or "Velvet Harbor watcher answered" when the work went to an agent-to-agent (A2A) partner. Their runs are in [Review what agents did](/xtm-one/use/work-history/).
- **An answer cut by a restart**: when XTM One restarts, is updated or recycles a worker during an answer, the answer resumes by itself as soon as XTM One is back, from the tool calls it had completed. In the transcript, one card stands for each cut: what interrupted the answer, the number of tool calls completed before the cut, and whether it is resuming by itself, resumed by itself, resumed by you or not resumed. Click the number of tool calls to list them by tool. While the answer resumes, the card shows it as the agent writes it, even after you reload the page.
- **An answer that could not resume by itself**: when the process stopped for another reason, such as a crash, or the same answer was cut again and again (3 times in a row without progress, 12 hours after its first cut, or 100 times in all), its card turns into a warning with **Resume** and **Dismiss**. **Resume** sends `Continue where you left off.`: the tool calls already completed are kept, so the agent goes on from their results. **Dismiss** hides the warning. An answer you stopped yourself, or one that ended more than 24 hours ago, starts again from scratch.
- **An answer cut before any of it was saved**: a notice at the end of the conversation takes the card's place. It says **It is resuming by itself: no need to click anything.**, or **This response stopped before it finished** with the same **Resume** and **Dismiss**.
- **A refused message**: a message the platform refuses gets no answer, and the chat shows the reason. When your session has ended, XTM One signs you out and tells you why.
- **The server out of reach**: during an update or a restart, the page says it reconnects by itself as soon as the server answers, and comes back without a click.
- **A new version**: after an update, a tab left open shows **A new version is available** when you come back to it. Click **Reload**.

While an agent works on a long request, a small game appears under the conversation, or in a corner of the window that shows the agent's reasoning; **Take control** opens it. Your answer comes first: a banner tells you when it is ready. An administrator can turn the game off with `ARCADE_ENABLED`, see the [configuration reference](/reference/xtm-one/configuration/).

:::example[Get the context of a night-shift alert in two voices]
At 02:40 on 12 April 2026, Paul Garnier, SOC analyst on the night shift at Grand Meridian Hotels, sees an EDR alert on a front-desk workstation of the Paris hotel: `powershell.exe` started by `outlook.exe`. He opens :menu[Chat], chooses **Night shift triage**, clicks **New conversation**, attaches the alert export `edr-alert-PAR-FD-03.json` with the paperclip, and writes:

> Does this match what Velvet Harbor did in January? @velvet-harbor-watcher add what the CTI team knows about their loader.

**Night shift triage** answers first: it reads the alert, finds in OpenCTI the January intrusion and its PowerShell loader, and sets out the matching steps. **Velvet Harbor watcher** answers next, in its own message, with the indicators of the last report on Velvet Harbor. Paul asks for "a CSV of the indicators to block, one per line": the CSV arrives as a card under the answer. The rest of the night is about Velvet Harbor, so he opens **Conversation options** > **Continue with another agent** and picks **Velvet Harbor watcher**, which answers his next questions with the whole exchange in view. He then opens **Conversation options** > **Continue in Slack or Teams** to follow up in a direct message with the Grand Meridian Slack bot from his phone. The first two answers counted as two agentic executions.
:::

## What's next

- [Share a workspace with your team](/xtm-one/use/workspaces/), to keep an investigation's conversations together.
- [Let an agent act later, at a time you choose](/xtm-one/use/work-items/), to have an agent post or check something at a set time.
- [Turn an agent's work into a presentation](/xtm-one/use/presentations/).
- [Set up your profile and API keys](/xtm-one/use/profile/), to choose your default agent and the General Assistant's tools.
