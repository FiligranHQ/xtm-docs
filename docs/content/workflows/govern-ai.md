---
title: Govern how AI is used across the platform
description: From XTM One, decide who signs in, what each agent reaches in OpenCTI, OpenAEV and OpenCRQ, which tools wait for a person, and keep one switch that stops every agent.
doc_type: journey
journey: govern-ai
products: [xtm-one, opencti, openaev, opencrq]
personas: [admin, executive, auditor]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 71
---

At the end of this journey, AI in your platform is under control: only your people sign in, each team reaches only the agents and secrets meant for it, prompts are screened for injection, a tool that changes something waits for a person, each AI feature of OpenCTI, OpenAEV and OpenCRQ is answered by the agent you chose, one switch stops every agent, and you can show what AI costs and who did what. An XTM One administrator drives it; the CISO and the auditors read the result. Every setting lives in XTM One, and applies to the AI of every registered product. The steps below are the pages to follow, in order.

::diagram{id=journey-govern-ai}

## Who takes part

- **The administrator** (Hélène Marchetti, CISO of Grand Meridian Hotels and administrator of XTM One) drives every step: edition, sign-in, groups, approvals, the prompt guard, intents, vaults, the emergency stop.
- **The platform engineer** (Marc Dupuis, IT operations) keeps the license, the session bounds and the vault in the deployment, and holds the printed emergency stop procedure on call.
- **The AI agent builder** (Sophie Laurent) sets her agents to ask before running tools, and shares the agent an intent should reach.
- **The SOC analyst** (Paul Garnier, on the night shift) answers the approvals of the runs made for him, and can stop his own activity.
- **The CISO and the auditors** read what AI costs per team and who changed what, in the chat with Platform Insights and in :menu[Administration > Audit logs].

## Why it spans several products

Once OpenCTI, OpenAEV and OpenCRQ are registered with XTM One, their AI runs in XTM One: each AI feature asks XTM One for an intent, such as `cti.container_summary` for the summary of a report, and an XTM One agent answers. The rules you set in XTM One therefore apply to the AI of every product.

- **What they send to XTM One.** At each registration, every product declares its intents and its vertical (`cti`, `aev`, `crq`). OpenCTI and OpenAEV also send their license, and a registered platform in Enterprise Edition puts XTM One in Enterprise Edition. OpenCRQ holds no license: its edition follows the XTM license XTM One returns. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **What XTM One decides for them.** Which agent answers each intent, per vertical; which tools ask before they run, with a switch per connected platform for the tools that change it; how many agent runs each person may spend; and, with the emergency stop of the whole platform, whether any AI runs at all, the autonomous attack paths XTM One drives for OpenAEV included.
- **What stays in each product.** An agent reaches a product with a token signed for the person it works for, so it never exceeds that person's capabilities, markings or grants there. And each product keeps its own consent: **Ask Ariane** stays off in OpenCTI and OpenAEV until the Filigran AI Terms are accepted. See [What an agent can reach, and with which rights](/xtm-one/use/in-opencti/#what-an-agent-can-reach-and-with-which-rights).

Set in each product alone, the same rules would drift apart, and none of them could stop an agent that runs in XTM One.

## What you need

- **XTM One, and which part is Enterprise Edition.** Sign-in, single sign-on, users and groups, secret vaults, :menu[Administration > Settings], :menu[Administration > Audit logs] and the security audit work in the Community Edition, and the emergency stop works in every case, AI off included. The rest is Enterprise Edition with AI switched on: intents, the agents whose tool calls approvals gate, connecting the Nova prompt guard (a new integration), the quotas of users and groups, and Platform Insights, a tool an agent answers with. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **OpenCTI, OpenAEV and OpenCRQ registered with XTM One.** See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one).
- **An identity provider** with an application for XTM One (OpenID Connect) or its SAML metadata, and **a vault** (HashiCorp Vault, AWS Secrets Manager, Google Cloud Secret Manager, Azure Key Vault or 1Password) with an identity for XTM One.
- **In XTM One**, the **Administrator** role for every setting of this journey. Whoever manages an agent sets its own **Ask before running tools**. A group manager reads the AI usage of their group's members in **Model usage**, and nothing more.
- **In OpenCTI**, the **Manage parameters** capability to accept the Filigran AI Terms. **In OpenAEV**, **Manage platform settings** for the same.

## How it unfolds

The story is the one of the step pages: in April 2026, after Velvet Harbor's fake summit booking reached the front desks of Grand Meridian Hotels in January, three teams use XTM One agents, and Hélène must be able to answer for each of them.

### Know what you are allowed to run

XTM One went into Enterprise Edition as soon as Grand Meridian's OpenCTI, in Enterprise Edition, registered, under the included allowance of 30 agentic executions a day. When the SOC used it up by mid-morning, Hélène installed an XTM One license with OpenAEV and OpenCRQ as **Sub-Licenses**; Marc keeps it in `ENTERPRISE_LICENSE`, on the platform and the worker. See [Know what your edition brings](/xtm-one/administer/editions/).

### Let only your people in, each with their team

Hélène connects Microsoft Entra ID through OpenID Connect, with **Allowed email domains** set to `grand-meridian.example`, **Automatically admin users** off, and **Local Authentication** kept on for the bootstrap account. A contractor outside the domain is refused. Marc bounds the sessions to a day, with 30 minutes of idle time. See [Sign in with your identity provider](/xtm-one/administer/sso/).

XTM One has two roles and no finer permissions: sharing decides what a person reaches. Hélène creates the groups **CTI team**, **SOC** and **Crisis cell**, makes Julien Bernard, the SOC manager, a manager of **SOC** so he follows its AI costs, and keeps **Crisis briefer** company-managed with **Restrict access** on **Crisis cell**. See [Give access with groups](/xtm-one/concepts/access/).

### Make risky tools wait for a person

Tool approvals are off until you turn them on. Sophie sets **Night shift triage** to **Ask**; in :menu[Administration > Settings > Access and security > Tool approvals], Hélène checks that the reads of OpenCTI and OpenAEV run without asking while the creation of a Jira ticket asks first, and makes `code_interpreter` ask too. At 02:40, Paul approves in :menu[Work > Operations] the ticket the agent proposes; its OpenCTI searches never stopped. **Approval for connected platforms** decides, for each registered product, whether its tools that change data ask. See [Approve what agents do](/xtm-one/administer/ai-governance/).

### Screen what reaches the agents

Some OpenCTI reports quote the fake booking request word for word: a text written to manipulate its reader, which **Velvet Harbor watcher** reads. Hélène connects **Nova Prompt Safety**, company-managed, keeps the community rules on **Log** for a week, then switches to **Block** those whose matches were all attacks, and screens the results of the web fetch tool. Each match is a `security_alert` entry in **Audit logs**. See [Protect agents from prompt injection](/xtm-one/administer/prompt-safety/).

### Decide which agent answers in each product

The **Summarize** of OpenCTI's reports is bound, built in, to **OpenCTI Entity Insights**. Hélène binds **Velvet Harbor watcher**, shared with **CTI team**, to `cti.container_summary` for the `cti` vertical, above the built-in agent. When Camille Roux summarizes a Velvet Harbor report in OpenCTI, the watcher answers; OpenAEV and OpenCRQ, on other verticals, are unchanged. See [Choose which agent answers](/xtm-one/administer/intents/).

### Keep the secrets out of XTM One

Hélène connects the Grand Meridian HashiCorp Vault under the reference name `corp`, granted to **SOC** on `secret/xtm-one/soc` only. The Jira token of **Night shift triage** becomes `vault://corp/secret/xtm-one/soc/jira#api_token`, and the Entra ID client secret is read the same way. A rotated token is used within five minutes, with nothing to change in XTM One. See [Keep secrets in your vault](/xtm-one/administer/secret-vaults/).

### Stop everything when it goes wrong

Hélène generates the emergency stop token and prints the procedure for Marc's on-call folder. On the night of 12 April, **Night shift triage** creates Jira tickets in a loop from an assignment that runs as Sophie, who had answered **Yes, always** to the Jira tool while testing: her saved approval covers the runs that act as her. Paul's own stop does not reach that assignment; Marc's command with the token stops the whole platform, and Hélène lifts it in the morning with a reason, once Sophie has fixed the agent. See [Stop every agent at once](/xtm-one/administer/emergency-stop/).

### Show what AI costs and who did what

Preparing next year's budget, Hélène asks the General Assistant, which carries Platform Insights for administrators, for the monthly cost of AI per group since January, projected to the end of 2027, then who changed the run budget of **Velvet Harbor watcher**. The answer reads the same figures as **Model usage** and the audit log, and warns that a person in two groups counts in each. See [Watch how AI is used](/xtm-one/use/platform-insights/).

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Build an AI agent your team can trust](/workflows/build-an-agent/): the agents this journey governs, built with only the tools they need.
- [Let the night shift ask agents from Slack](/workflows/answer-from-slack/): where approvals cannot be shown, and what a bot may run without asking.
- [XTM One in OpenCTI](/xtm-one/use/in-opencti/#govern-ai-on-the-platform), [in OpenAEV](/xtm-one/use/in-openaev/) and [in OpenCRQ](/xtm-one/use/in-opencrq/): what each product shows of these settings.
- [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/): keep AI off without cancelling anything.
