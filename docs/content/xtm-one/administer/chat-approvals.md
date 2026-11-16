---
title: Collect tool approvals in your own chat client
description: Let an embedded chat or your own application show the approval prompt of an XTM One agent and send the decision back, through a proxy that keeps the turn alive.
doc_type: how-to
products:
  - xtm-one
personas: [developer, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
no_screenshot: The task happens in the code of a client application and its proxy, with no XTM One screen.
migrated_from:
  - xtm-one:/deployment/chat-approvals-in-clients/
---

An XTM One agent that asks before running tools stops mid-answer and waits for a person to approve the tool call. In the XTM One web chat, the prompt shows by itself. A chat driven from outside, such as the assistant embedded in OpenCTI or OpenAEV or your own application, receives the prompt only if it says it can show one. This page opts a client in, lists what a proxy between the browser and XTM One must do, and says what happens in clients that cannot ask, Slack and Teams included. What asks first, and how administrators set it, is in [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/#approve-what-agents-do).

::diagram{id=xtm-one-administer-chat-approvals-proxy}

## What a client that cannot ask gets

A client that does not declare support never hangs: when a gated tool comes up, the turn ends with an ordinary answer. The agent names the tool, says it cannot collect the approval in this chat and stopped rather than acting without it, and points to the two ways forward: allow the tool for yourself, or ask an administrator to allow it for everyone.

The bots of Slack, Microsoft Teams and the other messaging apps are such clients: they cannot show a prompt. Their tools are never trusted automatically. Each bot has an **Ask before running tools** switch in :menu[Administration > Settings > Integrations > Channels & Bots]: turned off, conversations through that bot run their tools without asking. Another way is for the person to answer **Yes, always** once in the XTM One web chat: their standing approval then covers that tool in the conversations where they write with their own account.

## Before you start

- Your client talks to the platform chat API, `/api/v1/platform/chat/`, directly or through the backend of a suite platform. See [XTM One chat API](/develop/apis/chat/).
- It can show a prompt in the middle of a streamed answer, and send a request while the stream stays open.

## Opt a client in

:::steps

1. Send `"supports_tool_approval": true` in the body of each message, `POST /api/v1/platform/chat/messages`.
2. Read the stream. When a tool needs approval, XTM One pauses the turn and sends an `approval_required` event with the `conversation_id` and its `proposals`: for each call, the tool, what it does, the arguments and the schema they come from.
3. Show each argument with its description, not only its value: `cascade: true` says nothing, "cascade: also delete linked entities" says everything.
4. Send one decision per proposed call to `POST /api/v1/platform/chat/messages/approve`: `approve`, `reject` with a `rejection_reason` the agent can act on, or `approve_always`. A partial set is refused.
5. Keep reading the stream: the same turn goes on with the tool result, with nothing lost.

:::

`approve_always` saves a preference for that person, which also covers their scheduled runs: say so before they confirm. A decision carries no arguments: to correct a call, reject it with a reason.

### When the page reloads

The prompt arrives once, on the stream. On load, ask for it again:

```
GET /api/v1/platform/chat/conversations/{conversation_id}/pending-approvals
```

An empty list is the usual answer. Otherwise show the same prompt and send the same decision. The reply then goes to the conversation, not to a stream: while the response reads `"turn": "running"`, keep polling and show your working indicator, then fetch the conversation once it reads `"idle"`. A turn nobody comes back to (no request about the conversation for 30 minutes) stops waiting; a reviewer who is only thinking is never on a clock.

## What a proxy must do

The assistant embedded in a suite platform rarely reaches XTM One directly: the platform's backend authenticates the browser, signs a platform token and forwards the call. A proxy of your own does the same. Four things decide whether approvals work through it:

- **Forward the whole body.** A proxy that rebuilds the body from a fixed list of fields drops `supports_tool_approval`: XTM One then never pauses, and the client gets the plain answer.
- **Route the decision back.** Forward `messages/approve` next to `messages/steer`, while the stream stays open.
- **Never time out a streaming turn.** A paused turn stays silent for as long as the reviewer takes: any read timeout on the way becomes an approval deadline nobody chose. Set the streaming call's timeout to none, at every hop, ingress included.
- **Pass keepalives through.** XTM One sends a `: keepalive` comment line every 15 seconds during silent windows. Stream bytes through rather than parsing and re-emitting the events you know.

Accept decisions only from a real user session, with the same check as your chat route, and refuse agent and service identities on that route: an agent approving its own tool call defeats the control. When the session expired during a long pause, tell the reviewer to sign in again and keep the prompt: after the refresh, `pending-approvals` restores it.

## Connected platforms that cannot ask

A suite platform that registers with XTM One brings its own tools. Their reads run without asking; the tools that change the platform ask. A platform whose assistant can show prompts says so when it registers, with `supports_approval_prompts: true`. For one that cannot yet, an administrator decides in :menu[Administration > Settings > Access and security > Tool approvals], **Approval for connected platforms**: **Require approval**, or let its tools run. `TOOL_APPROVALS_PLATFORM_DEFAULT=skip` sets that choice for the platforms registering for the first time; it never overwrites an administrator's choice.

That choice covers the platform's own tools, not XTM One's. For those, an administrator sets **Ask before running tools** to **Don't ask** on the agents the assistant drives (the built-in ones included), or lets the tool run without asking for everyone.

:::example[Approve a Jira ticket from the Grand Meridian SOC portal]
The SOC of Grand Meridian Hotels chats with "Night shift triage" from its internal portal, whose backend forwards the chat to `https://xtm-one.grand-meridian.example` with a platform token. The agent asks before running tools, and creating a Jira ticket asks first.

The portal sends each message with the flag:

```json
{
  "conversation_id": "5b2f0c1e-8d4a-4f6b-9a3e-1c7d2e9f0a41",
  "agent_slug": "night-shift-triage",
  "content": "Open a ticket for the PowerShell loader alert on the front-desk PC of the Lyon hotel",
  "stream": true,
  "supports_tool_approval": true
}
```

The stream pauses on an `approval_required` event proposing the Jira tool with its summary, project and priority. Paul Garnier reads the arguments and approves; the portal sends:

```json
{
  "conversation_id": "5b2f0c1e-8d4a-4f6b-9a3e-1c7d2e9f0a41",
  "decisions": [
    {"tool_call_id": "call_7Qk2Lm", "decision": "approve"}
  ]
}
```

The same answer resumes with the ticket's key. The portal's ingress was set with no read timeout on the chat route, so the pause of two minutes cost nothing.
:::

## What's next

- [XTM One chat API](/develop/apis/chat/)
- [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/)
- [Talk to agents from your messaging apps](/xtm-one/administer/channels/)
