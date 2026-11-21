---
title: Start an agent from another tool with a webhook
description: Let a SIEM, a SOAR playbook or a ticketing tool start an XTM One agent's assignment by posting to its webhook URL with its secret, and diagnose a delivery that does not arrive.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, soc-manager]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/webhook-triggers/
---

A webhook trigger lets another system start an AI agent's work. Your SIEM raises an alert, your SOAR runs a playbook, your ticketing tool opens a case: each one posts to a URL, and the agent starts working on what it received. The URL belongs to an assignment, the automation rule that tells an agent when to run and what to do, with **Webhook** as its trigger type. This page finds the URL and its secret, authenticates the request, wires a SOAR playbook, and reads the answer when a delivery does not arrive. Creating the agent and its assignment is in [Create an agent](/xtm-one/build/agents/).

::diagram{id=xtm-one-webhook-triggers-delivery}

## Before you start

- The agent has an assignment whose **Trigger type** is **Webhook**, and the assignment is enabled.
- You can manage the assignment: only people who can manage it see its secret. If the **Trigger** card shows the URL but no secret, ask the assignment's owner.
- The sending system can reach XTM One over HTTPS and set a request header.

## Find the URL and the secret

:::steps
1. Open :menu[Orchestration > Agents] and the agent, then open the assignment from its **Assignments** card.
2. Read the **Trigger** card:
   - **Webhook URL**: the address the sending system posts to. It already holds the assignment's identifier, under `/api/hooks/assignments/`.
   - **Webhook secret**: the credential the sending system presents. Every webhook assignment gets one when it is created.
   - **How to trigger**: a ready-made `curl` request with both, to test the assignment before you touch the other system.
3. Copy the **How to trigger** request and run it from a terminal. A run of the agent appears in :menu[Work > Operations].

::screenshot{id=xtm-one-webhook-triggers-trigger-card alt="The Trigger card of a webhook assignment: the Webhook trigger type, Exact event matching, the Webhook URL, the Webhook secret to send with each request, and How to trigger."}
:::

The agent receives the JSON body of the request with the assignment's instructions. Send the fields the agent needs to work, such as the alert's name, its host and its identifier, rather than a link it cannot open.

## Authenticate the request

Send the secret in one of these headers. A request with none of them is refused with `401`, even when the URL and the secret are right: check the header's name first.

| Header | Value |
| --- | --- |
| `X-Webhook-Token` | The secret, as it is. The first choice for any new sender |
| `X-Webhook-Secret` | The same, under the name the **Webhook secret** label suggests |
| `Authorization` | `Bearer` followed by a space and the secret, for senders that only offer standard authentication fields |
| `X-OpenCTI-Token` | The original name, still accepted so that existing senders keep working. Use one of the names above for anything new |

A sender that can compute an HMAC can sign the body instead and keep the secret off the wire: put the hexadecimal HMAC-SHA256 of the raw body, keyed with the secret, in `X-Signature`. The signatures of GitHub and Slack webhooks are recognized too.

:::warning
Anyone who has the secret can start the assignment with any body until someone changes it. Keep it in the sending system's credential store and reference it there, never as plain text in a playbook that is exported, cloned or committed. A signed body (`X-Signature`) never sends the secret at all: prefer it when the sender can compute one.
:::

An administrator can refuse every delivery to an assignment without a secret, platform-wide, with `WEBHOOK_REQUIRE_SECRET`: see the [configuration reference](/reference/xtm-one/configuration/).

## Trigger from a SOAR playbook

Many SOAR webhook actions only offer a URL and a token field, with nowhere to put a header. Use the playbook's generic HTTP request action instead. In Google SecOps SOAR, it is **Execute HTTP request**:

| Field | Value |
| --- | --- |
| `Method` | `POST` |
| `URL Path` | The **Webhook URL** of the **Trigger** card |
| `Headers` | `{"Content-Type": "application/json", "X-Webhook-Token": "[the secret, from the credential store]"}` |
| `Body Payload` | The JSON the agent receives, such as `{"case_id": "[Case.case_id]", "alert_name": "[Alert.alert_name]"}` |
| `Fail on 4xx/5xx` | `true`, so a refused or throttled delivery shows in the playbook instead of passing for a success |
| `Request Timeout` | `120` |

Write plain JSON in `Headers` and `Body Payload`: escaped quotes (`{\"Content-Type\": ...}`) make the playbook reject the value before sending it. Keep each placeholder inside its quotes, as in `"[Alert.alert_name]"`.

## Choose the events a trigger runs on

A trigger can select events: an integration event such as GitHub `pull_request`, or the `events` list of a webhook trigger. It runs on exactly the events it selects: `pull_request` runs on `pull_request.opened` and `pull_request.closed`, never on `pull_request_review`; an `events` entry `issues.opened` never matches `issues.opened_by_bot`.

A trigger saved before exact matching existed keeps the prefix match it had, so that nothing that ran before stops: the **Trigger** card shows it as **Legacy event matching**, with the events it also runs on. Click **Use exact matching** to switch it, or choose its events again in the trigger editor. An administrator applies exact matching to every trigger at once with **Exact event matching for every trigger**, in :menu[Administration > Settings > Access and security > Security options], or with `WEBHOOK_EXACT_EVENT_MATCHING=true`.

## When a delivery does not arrive

Read the answer the sending system received:

| Answer | Meaning | What to do |
| --- | --- | --- |
| `401 Missing webhook signature…` | No accepted credential header | Check the header's name; the message lists the accepted ones |
| `401 Invalid webhook token` | The header is right, its value is not | Copy the secret again from the **Trigger** card |
| `401 Invalid webhook bearer token` | Same, for `Authorization: Bearer` | Copy the secret again |
| `401 Invalid webhook signature` | The body was signed with another secret, or changed after signing | Sign the exact raw body with the current secret |
| `403` | XTM One is not in the Enterprise Edition: AI work does not start | See [What each edition of XTM One brings](/xtm-one/administer/editions/) |
| `429` | Too many deliveries, or too many waiting | Retry after the delay the answer gives |
| `"status": "ignored"` | Accepted, but nothing matched: the assignment is disabled, deleted, or not a webhook assignment | Check the assignment |
| `"reason": "event not selected"` | A GitHub delivery for an event the trigger does not select | Check the events on the **Trigger** card and in GitHub |

Every run that started is in :menu[Work > Operations], with runs still waiting behind an active one: see [Review what agents did](/xtm-one/use/work-history/). An assignment runs up to 10 deliveries at once by default (`ASSIGNMENT_MAX_CONCURRENT_RUNS`); a delivery that arrives while they all run waits in the queue and starts as soon as one ends. Identical deliveries within a short window are counted once. The limits of delivery, the queue and the de-duplication window are in the **Webhook delivery** section of the [configuration reference](/reference/xtm-one/configuration/).

:::example[Brief the CTI team when the SOAR opens a Velvet Harbor case]
Grand Meridian Hotels' SOAR opens a case each time the EDR blocks a PowerShell loader on a front-desk workstation. Sophie Laurent gives **Velvet Harbor watcher** an assignment `SOAR case to CTI brief`, with **Trigger type** **Webhook** and the instruction "Compare the case with what OpenCTI knows about Velvet Harbor and post a short brief for the CTI team". She stores the **Webhook secret** in the SOAR's credential store as `xtm_one_velvet_harbor_secret`, and configures the playbook's HTTP request action:

| Field | Value |
| --- | --- |
| `Method` | `POST` |
| `URL Path` | The **Webhook URL**, on `https://xtm-one.grand-meridian.example/api/hooks/assignments/` |
| `Headers` | `{"Content-Type": "application/json", "X-Webhook-Token": "[Credentials.xtm_one_velvet_harbor_secret]"}` |
| `Body Payload` | `{"case_id": "[Case.case_id]", "alert_name": "[Alert.alert_name]", "host": "[Alert.host]"}` |
| `Fail on 4xx/5xx` | `true` |

On 12 April 2026, the EDR blocks a loader on `PAR-FD-03`: the playbook posts the case, the agent starts, and its run in :menu[Work > Operations] shows the brief it sent to the CTI team. The first test had returned `401 Missing webhook signature…`: the header had been typed `X-Webhook-Key`.
:::

## What's next

- [Review what agents did](/xtm-one/use/work-history/), to read the run a delivery started.
- [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/): the **Schedule** view of its map lists event-driven assignments.
- [Set what one agent run may spend](/xtm-one/use/run-budgets/), to bound the cost of each delivery.
