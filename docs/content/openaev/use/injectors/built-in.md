---
title: Built-in injectors and their injects
description: The injectors OpenAEV ships with, the inject types each one adds to your scenarios, the fields of each inject type, and the settings each one needs.
doc_type: reference
products:
  - openaev
personas: [exercise-planner, validation-engineer, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/injects-builtin/
---

OpenAEV ships with injectors that need no deployment. Six run inside the platform from its first start and cover what most scenarios need: emails, media pressure, challenges, manual actions, phishing, and attack actions on endpoints. Two more ship with the platform but are deployed from the catalog: OpenCTI and OVHCloud SMS Platform. This page lists each one, the inject types (contracts) it adds, the fields of each inject, and the settings it needs. How injectors work, and the external ones, is in [How injectors deliver your injects](/openaev/use/injectors/).

::diagram{id=openaev-built-in-injectors-map}

## Always present

| Injector | Inject types | Delivers to | Needs |
| --- | --- | --- | --- |
| **Email** | **Send individual mails**, **Send multi-recipients mail** | Players of the target teams | The mail server of the platform |
| **Media pressure** | **Publish a media pressure** | Players, on the player interface | Channels and articles, in :menu[Components > Channels] |
| **Challenges** | **Publish challenges** | Players, on the player interface | Challenges, in :menu[Components > Challenges] |
| **Manual** | **Manual** | The animation team, as a reminder | Nothing |
| **Phishing** | One per phishing landing page | Players of the target teams | The mail server of the platform, and landing pages in :menu[Components > Phishing] |
| **OpenAEV Implant** | One per action of the Threat Arsenal | Endpoints and asset groups | An executor on the targets: the OpenAEV agent or your EDR |

The **Integrations** list shows these injectors with **Runs inside the platform**: you cannot stop or delete them.

### Email

Sends emails to the players of the target teams, and to extra addresses typed in **Recipients**.

| Inject type | What it sends | Fields |
| --- | --- | --- |
| **Send individual mails** | One email per player, so each one reads their own | **Subject** and **Body** (required), **Recipients**, **Encrypted**, attachments, expectations |
| **Send multi-recipients mail** | One email to all the players at once, each one seeing the others | **Subject** and **Body** (required), **Recipients**, attachments, expectations |

The platform sends through its mail server, set by the [`spring.mail` block](/reference/openaev/configuration/#spring-mail) of the configuration. The sender is `openaev.default-mailer`, or the username of the IMAP account when the platform reads replies from a mailbox and that username is an email address, set by the [`openaev.mail` block](/reference/openaev/configuration/#openaev-mail). An IMAP username that is a plain login, such as `openaev`, leaves `openaev.default-mailer` as the sender.

### Media pressure

**Publish a media pressure** publishes articles on the player interface, to simulate the news coverage of a crisis. Its fields are the target teams and the articles (required), attachments, expectations, and **Send email** (on by default), which also tells the players by email. Articles belong to a channel, which gives them their look. See [Media pressure](/openaev/use/components/media-pressure/).

### Challenges

**Publish challenges** gives the players of the target teams challenges to solve, such as a flag to find in a file, and tells them by email. Its fields are the challenges and the target teams (required), the email's **Subject**, by default `New challenges published for ${user.email}`, and **Body** (required), **Encrypted**, attachments and expectations. See [Challenges](/openaev/use/components/challenges/).

### Manual

**Manual** is a reminder for the animation team: an action they carry out outside the platform, such as a phone call from a fake journalist. It sends nothing. When you add expectations to it, you choose the teams whose response the animation team will score by hand.

### Phishing

Sends a credential-harvesting email to each player of the target teams, with a link to a landing page that OpenAEV hosts, and scores who opens, clicks and submits. Each landing page adds its own inject type to the Threat Arsenal. It uses the same mail server as the email injector. See [How a phishing exercise works](/openaev/use/phishing/).

### OpenAEV Implant

Runs the actions of the Threat Arsenal on endpoints: commands, executables, file drops, DNS resolutions and the other kinds of actions. Each action is an inject type of this injector. The injector does not reach the endpoints itself: an executor starts the OpenAEV implant on each target, and the implant runs the action and reports. It needs an executor on every target, see [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/), and the actions are described in [Threat Arsenal](/openaev/use/threat-arsenal/).

::screenshot{id=openaev-injector-email-contracts alt="The Email injector, started and built in, with its two inject types, Send individual mails and Send multi-recipients mail."}

## Shipped with the platform, deployed from the catalog

These two run inside the platform too, but only once you deploy them from :menu[Integrations], **Available** tab. Deploying them requires an Enterprise Edition license and the **Manage tenant settings** capability. A platform that had them enabled in its configuration before turns them into deployed injectors at its first start.

| Injector | Inject types | Fields | Needs |
| --- | --- | --- | --- |
| **OpenCTI** | **Create a new case**, **Create a new report** | **Name** and **Description** (required), attachments, expectations | A connection to OpenCTI, in the [`openaev.xtm` block](/reference/openaev/configuration/#openaev-xtm) |
| **OVHCloud SMS Platform** | **Send a SMS** | The target teams and **Message** (required), expectations | An OVHcloud SMS account: its keys, service and sender go in the injector's form |

The OpenCTI injector creates the case or the report in OpenCTI, so that the CERT works the incident of the exercise in its own tool. The SMS injector texts the phone number of each player of the target teams.

:::example[The injects of Grand Meridian's crisis exercise]
For the crisis cell of Grand Meridian Hotels, the exercise planner turns the night of 9 January 2026 into the tabletop exercise **Velvet Harbor crisis night**, built from built-in injectors only. It starts at 22:00:

| Time | Inject type | Content |
| --- | --- | --- |
| 22:40 | **Send individual mails** | `Call the crisis cell to the bridge`: **Subject** `${hotel_group}: crisis cell on the bridge now`, to the team **Crisis cell**, signed by the CISO Hélène Marchetti |
| 22:50 | **Publish challenges** | `Which account was used?`: find, in the exported EDR alert, the account the loader ran under: the answer is `night.audit` |
| 23:30 | **Publish a media pressure** | The article `Guest records of Grand Meridian Hotels offered on a criminal forum`, on the channel `Hotel Business Daily`, with **Send email** on |
| 23:45 | **Manual** | A journalist calls the Paris head office: the animation team plays the call and scores the answer |

None of them needs a deployment: only the platform's mail server must be set.
:::

:::xtm-one[Write the injects with XTM One]
When OpenAEV is connected to XTM One, AI agents can write the content of these injects: the messages of an email inject and the articles of a media pressure inject, from a short description of the situation. Connecting OpenAEV to XTM One is in [Connect the modules](/deploy/configure/connect-modules/).
:::

## What's next

- [How injectors deliver your injects](/openaev/use/injectors/): the kinds of injectors and the external ones.
- [Media pressure](/openaev/use/components/media-pressure/) and [Challenges](/openaev/use/components/challenges/): build the content these injectors publish.
- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/): the executor the OpenAEV Implant needs.
