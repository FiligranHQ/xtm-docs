---
title: Let the night shift ask agents from Slack
description: Connect Slack to XTM One so the SOC asks an AI agent from its own channel and gets answers from OpenCTI and OpenAEV, with only the tools an administrator lets it run there.
doc_type: journey
journey: answer-from-slack
products: [xtm-one, opencti, openaev]
personas: [admin, soc-analyst, ai-builder]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 73
---

At the end of this journey, your SOC asks an XTM One AI agent a question in the Slack channel it already works in, and the agent answers in the thread with what OpenCTI knows of the threat and what OpenAEV's last simulation showed, read with the rights of the analyst who asked. Tools that change something do not run from Slack unless an administrator allows it. An XTM One administrator connects Slack and decides what the bot may do, the AI agent builder turns Slack on for the agent, and the SOC analysts use it. The steps below are the pages to follow, in order.

::diagram{id=journey-answer-from-slack}

## Who takes part

- **The administrator** (Hélène Marchetti, CISO of Grand Meridian Hotels and administrator of XTM One) drives the journey: she creates the Slack app, connects the bot, narrows the channels it hears, and decides what runs without asking.
- **The AI agent builder** (Sophie Laurent) turns Slack on for the **Night shift triage** agent and restricts it to the SOC's channel.
- **The SOC analysts** (Paul Garnier, on the night shift) ask the agent from `#soc-night`, and answer in XTM One the approvals Slack cannot show.

## Why it spans several products

The question is asked in Slack; the answer is in OpenCTI and OpenAEV; XTM One connects them.

- **XTM One** holds the Slack bot, decides which agent answers in which channel, runs the agent, and applies the tool approvals. See [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- **OpenCTI** knows the threat: whether 203.0.113.10 is an Indicator of Velvet Harbor, and in which intrusion it was seen. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
- **OpenAEV** knows the defenses: what the last simulation of Velvet Harbor's techniques on the front-desk workstations prevented and detected. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).

What passes between them: Slack sends the message and the sender's email to XTM One. XTM One matches the email to an account, and the agent calls OpenCTI and OpenAEV with a token signed for that same email, so each product applies that analyst's own capabilities, markings and grants. The answer goes back to the Slack thread. An approval never passes through Slack: it is answered in XTM One. Without the agent, the analyst would open both products in the middle of the night and cross-check by hand.

## What you need

- **XTM One in Enterprise Edition, with AI switched on.** Channel bots are an AI area: saving a new bot is refused otherwise. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **A public HTTPS address** for XTM One, its `BASE_URL`: Slack posts every event to the webhook URL built from it.
- **A Slack workspace** where you may create and install an app, or a Slack administrator who approves it.
- **OpenCTI and OpenAEV registered with XTM One**, and bound to the agent that answers. See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one) and [Create an agent](/xtm-one/build/agents/).
- **One email per analyst, everywhere.** Each analyst has an XTM One account with the email of their Slack profile, and the same email in OpenCTI and OpenAEV. A sender XTM One cannot match is refused, unless the bot allows anonymous users; even then, the agent reads nothing from OpenCTI or OpenAEV for them, since those integrations sign a token for a person.
- **In XTM One**, the **Administrator** role to connect the bot and set its switches; owning the agent, or **Can configure** on it, to turn Slack on for it.
- **In OpenCTI**, the **Access knowledge** capability for each analyst. **In OpenAEV**, **Access assessment**, to read simulations and their results.

## How it unfolds

The story is the one of the step pages: in April 2026, three months after Velvet Harbor reached the front desks of Grand Meridian Hotels, the night shift wants answers at 2 a.m. without leaving `#soc-night`.

### Decide how the bot will behave

Hélène first reads how a channel bot works: one bot per messaging app, shared by agents; **Receive** for the messages an agent answers and **Publish** for the ones it posts on its own; receive and publish whitelists that limit the channels; and the switches of the bot card, which no agent can loosen. She decides on one channel, `#soc-night`, one agent, and **Only when mentioned**. See [Understand channels](/xtm-one/administer/channels/).

### Connect Slack

In the Grand Meridian Slack workspace, Hélène creates the app **XTM One** with the thirteen bot scopes, `users:read.email` among them, so XTM One can tell who writes. She pastes its bot token and signing secret on the **Slack** card in :menu[Administration > Settings > Integrations > Channels & Bots], gives Slack the webhook URL ending with `/api/channels/slack/events`, subscribes to `message.im` and `app_mention`, and invites the bot to `#soc-night`. On the card, she sets the **Receive** whitelist to `#soc-night`. Sophie turns on **Slack** for **Night shift triage**, with **Receive** on and **Restrict to specific channels** set to `#soc-night`. See [Connect Slack](/xtm-one/administer/channels/slack/).

### Choose between a bot and an account

A bot speaks as itself: people mention **XTM One**. An agent account instead lets the agent answer from a real user account, but only on Microsoft Teams and Outlook, not on Slack. Grand Meridian keeps the bot in Slack, and gives the same agent a Teams account, **SOC Night Assistant**, for the SOC team's **Night shift** channel in Teams. See [Give the agent its own account](/xtm-one/administer/channels/agent-accounts/).

### Decide which tools run from Slack

Slack cannot show an approval prompt. **Night shift triage** asks before running tools, and the bot's **Ask before running tools** stays on, its default: its reads of OpenCTI and OpenAEV run, but a tool that waits for approval, such as creating a Jira ticket, ends the turn with a message that says so. Paul then asks for the ticket in the XTM One web chat, where the approval shows: a channel thread is never continued on the web, since other people read it. Turning the bot's switch off lets every conversation through it run its tools unattended: XTM One asks to confirm, and records it in :menu[Administration > Audit logs]. See [Approve what agents do](/xtm-one/administer/ai-governance/).

### Ask from the channel

At 02:14, the proxy of the Lyon hotel flags 203.0.113.10. Paul writes in `#soc-night`: "@XTM One is 203.0.113.10 linked to Velvet Harbor?". Slack gives XTM One his email, so the agent works as Paul and reads OpenCTI with his rights: the address is an Indicator of Velvet Harbor, seen in the intrusion of 9 January 2026. See [What it reads in OpenCTI](/xtm-one/use/in-opencti/).

In the same thread, without mentioning the bot again, Paul asks whether the front desks would stop that traffic. The agent reads the last simulation of Velvet Harbor's techniques in OpenAEV, again as Paul: Web Protocols was detected, but not prevented. Paul types `/stop` once the alert is closed. See [What it reads in OpenAEV](/xtm-one/use/in-openaev/).

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Build an AI agent your team can trust](/workflows/build-an-agent/): the agent that answers, with only the tools it needs.
- [Govern how AI is used across the platform](/workflows/govern-ai/): approvals, the prompt guard and the emergency stop that also apply to what the bot runs.
- [Check your defenses against the threats you track](/workflows/threat-informed-validation/): the simulation whose results the agent reads.
- [Get answers from an agent in the chat](/xtm-one/use/chat/#continue-in-slack-or-teams): carry a web conversation on in a Slack direct message.
