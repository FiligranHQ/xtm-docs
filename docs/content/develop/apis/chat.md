---
title: XTM One chat API
description: "Talk to an XTM One agent from your code over HTTP: send a message and read its answer as server-sent events, follow, stop, steer or approve a request, and know how every request ends."
doc_type: reference
products:
  - xtm-one
personas: [developer, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-12
edition: ee
no_screenshot: "This page describes HTTP routes and the server-sent events they stream: the chat screen that uses them is described on the chat page."
migrated_from:
  - xtm-one:/usage/chat-api/
---

The web interface of XTM One, its Office add-in and your own scripts talk to a conversation over HTTP. A message goes in a `POST` whose response streams the request back as server-sent events (SSE), and a second stream follows the rest of the conversation. This page is the reference of those routes, under `/api/v1/web/chat`. It covers how to sign in and open a conversation, what to send, and the frames you receive while an AI agent works. It also covers how to stop, steer or approve a request, and what you can count on when a request ends. The REST routes under `/api/v1/chat/conversations`, listed at the end, serve the same conversations without a stream.

:::note[The WebSocket is gone]
From XTM One 1.261112.0, these routes replace the WebSocket of a conversation, `/api/v1/chat/conversations/{conversation_id}/ws`, and its close codes. [XTM One 1.261112.0](/deploy/upgrade/breaking-changes/#xtm-one-1-261112-0) maps each WebSocket message to its route.
:::

::diagram{id=xtm-one-develop-chat-frames}

## Before you start

- **The Enterprise Edition, with AI on.** Without them, the routes that start or feed a request answer 403: see [Refusals](#refusals) and [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **A session token.** `POST /api/v1/auth/login` with `{"email": ..., "password": ...}` returns it as `access_token`. When the account has two-factor authentication, the answer carries `"mfa_required": true` and an `mfa_token` instead, which only `POST /api/v1/auth/mfa/verify` accepts. When local sign-in is turned off for single sign-on, the route answers 403. Send the token on every request as `Authorization: Bearer {access_token}`. The routes under `/api/v1/web` refuse a personal API key or a platform token with a 403, `The web chat needs a web session`; the REST routes accept a personal API key, `Authorization: Bearer fcp-...`.
- **A conversation you own.** Only its owner sends to it, follows it, and stops, steers or approves its requests. A conversation shared with you can be opened and paged, read-only.

Every message, streamed or over REST, counts against a limit of 30 a minute per person, and one agent run against the person's quota.

## Read a stream

Both streams are `text/event-stream` responses. Each event is one `data:` line that holds a JSON frame with a `type`, followed by a blank line:

```text
data: {"type": "stream", "content": "The loader of ", "turn_id": "c41f0e8a2b7d4c96a3e15f8d0b7c2e49"}

: keepalive

```

Once a stream has started, it writes a `: keepalive` comment at most 15 seconds after the last thing it wrote, so that proxies keep it open: skip it. Skip any frame `type` you do not know, too.

Name your client with an `X-Chat-Client-Id` header on every route that sends or acts on a request, up to 128 printable characters. Pass the same value as `client_id` when you [follow the conversation](#requests-you-did-not-send): your own requests are then never repeated there. XTM One replaces a missing or invalid value with a random one.

### Refusals

A refusal is an HTTP error with a readable `detail`. Where a client may need to branch on it, a `code` sits beside it:

| Status | Body | When |
| --- | --- | --- |
| 403 | `{"detail": "...", "code": "terms_required"}` | The person has not accepted the terms of use yet: they sign in once in the interface to accept them. Every route |
| 403 | `{"detail": "...", "code": "chat_unavailable"}` | The deployment is in the Community Edition. Only the routes that start or feed a request: send, steer, approve, upload, and open a new conversation. Restoring a conversation stays open |
| 403 | `{"detail": {"code": "ai_disabled", "message": "..."}}` | An administrator switched AI off. The same routes as `chat_unavailable` |
| 401 | `detail.code` is `session_expired` | The session ended. Sign in again |

Without chat, everything else keeps working: listing, reading, renaming and deleting conversations, the events stream, the pending approvals, stop and continue.

A stream still open when its session ends sends `{"type": "session_expired", "reason": "lifetime"}`, or `"idle"`, and ends. XTM One checks the session every 15 seconds. On the mobile app, a stream still open when the app locks behind its PIN sends `{"type": "app_locked"}` and ends.

## Open a conversation

`POST /api/v1/web/chat/sessions` with `{"agent_slug": "night-shift-triage"}`, or `agent_id`, creates a conversation and answers its `conversation_id`. Add `workspace_id` to file it into a workspace. Without an agent, the General Assistant answers it. An unknown slug answers 404, `Agent not found`.

With `{"conversation_id": ...}`, the same route restores that conversation instead; one you cannot read starts a new one. A restore answers:

| Field | What it holds |
| --- | --- |
| `messages`, `has_more` | The latest messages, oldest first; `has_more` is `true` when older ones exist |
| `agent_id`, `agent_name`, `workspace_id` | The conversation's agent and workspace. Without an agent, `agent_name` names who answers: the agent standing in for the General Assistant, or the General Assistant |
| `is_processing` | `true` while a request runs |
| `pending_approvals` | The tool calls a running request waits for you to [approve](#approve-a-tool-call), or `null` |
| `interrupted_turn` | The last request stopped before its answer, with `started_at` and `interrupted_at`, or `null`: see [How a request ends](#how-a-request-ends) |

The other routes of a conversation:

| Route | What it does |
| --- | --- |
| `GET /api/v1/web/chat/agents` | The agents you can chat with: `id`, `name`, `slug`, `icon`, `description`, `group`, `badges` and `disable_chat`. The General Assistant comes first, unless an agent stands in for it |
| `GET /api/v1/web/chat/sessions` | Your conversations, as `{"conversations": [...]}`. `limit` from 1 to 200, 50 by default, `offset`, `workspace_id`, and `scope=shared` for those shared with you |
| `GET /api/v1/web/chat/sessions/{conversation_id}/messages` | The messages older than the message `before`, as `{"messages": [...], "has_more": ...}`. `limit` up to 500, 50 by default |
| `PATCH /api/v1/web/chat/sessions/{conversation_id}` | `{"title": ...}` renames the conversation, `{"workspace_id": ...}` files it into a workspace |
| `DELETE /api/v1/web/chat/sessions/{conversation_id}` | Archives the conversation. Answers 204 |
| `DELETE /api/v1/web/chat/sessions` | Archives every conversation `GET /api/v1/web/chat/sessions` lists. Answers 204 |
| `POST /api/v1/web/chat/upload` | A multipart form with `conversation_id` and `file`: stores the file for the next message and answers `{"file_ids": [...]}` |

## Send a message

`POST /api/v1/web/chat/messages` sends a message and streams its request back:

```bash
curl -N https://xtm-one.grand-meridian.example/api/v1/web/chat/messages \
  -H "Authorization: Bearer $XTM_ONE_TOKEN" \
  -H "Content-Type: application/json" \
  -H "X-Chat-Client-Id: soc-siem-script" \
  -d '{"conversation_id": "9a7f2e1d-4c3b-4e5a-8f6d-1b2c3d4e5f60", "content": "Summarize the alerts of FD-PAR-03 since 22:00"}'
```

| Field | What it holds |
| --- | --- |
| `conversation_id` | The conversation, which must be yours |
| `content` | The message |
| `file_ids` | Files stored beforehand with `POST /api/v1/web/chat/upload` |
| `mentioned_agent_ids` | Agents that answer after the conversation's agent, in order: see [Mention several agents](#mention-several-agents) |
| `referenced_conversation_ids` | Other conversations the agent reads: see [Reference other conversations](#reference-other-conversations) |
| `supports_tool_approval` | `true` when your client [answers approvals](#approve-a-tool-call) |

A message refused before its request starts gets an HTTP error with `{"detail": ...}` instead of a stream:

| Status | Why |
| --- | --- |
| 400 | The message is empty, or it adds a tool, knowledge base or integration you may not use |
| 401 | The account may no longer chat, for example because it was deactivated |
| 404 | `Conversation not found`: no such conversation of yours |
| 409 | The conversation does not take a message from here: a group channel thread, or a request a channel message runs |
| 422 | The Deep Investigation choices could not be read |
| 429 | Too many messages: `Retry-After` says when to send again. A platform set to fail closed answers 503 while its rate limiter cannot be reached |

That is why the status and headers of the response arrive with its first frame past `thinking`: nothing is written before them, not even a keepalive. Once the request has started, a failure is an `error` frame.

The response ends once the request, and the new title of the conversation if any, is delivered. Closing it early does not stop the request, nor does reading nothing for about 10 seconds, after which XTM One ends the response. The request runs on and stores its answer.

## Frames

| `type` | Meaning |
| --- | --- |
| `thinking` | Your message was received. It comes before the request runs |
| `turn_started` | Your message started a request of its own; `turn_id` names it. A message folded into a running request receives `folded_into_active_turn` instead: see [Messages folded into a running request](#messages-folded-into-a-running-request) |
| `status` | Progress: `status` names the step, such as `preparing`, `tool_start`, `thinking_text` or `streaming`. `wait_start` announces a pause of the agent |
| `stream` | The next piece of the answer as it is written, in `content` |
| `done` | The answer: `content`, and `message_id`, the stored assistant message. With `"intermediate": true`, it is a part of the answer already delivered while the agent keeps working: the request is not over |
| `agentic_action` | The request was charged `actions` agentic actions, counted in the quota `period` it names |
| `approval_required` | The request waits for you to [approve](#approve-a-tool-call) tool calls, in `proposals`. `approval_resolved` says the wait is over |
| `conversation_title` | The conversation was given a title, in `title` |
| `error` | The request ended without an answer; `content` says why |
| `cancelled` | The request was stopped |

Every frame of the request carries its `turn_id`, `thinking` included; `conversation_title` belongs to no request and carries none. An `error` or `cancelled` with `"provisional": true` ends the request, but its stored end follows on the same stream. That end is a `done` or `cancelled` with `message_id`, or an `error` when the reply could not be stored. Any other `error` or `cancelled` is the last frame of the request.

## Stop, continue or steer a request

The other actions on a running request are `POST` routes with the same headers. Each takes `{"conversation_id": ...}`:

| Route | What it does |
| --- | --- |
| `/api/v1/web/chat/messages/cancel` | Stops the request and the background work it started. Answers 202, `{"status": "cancelled", "turn": ...}` |
| `/api/v1/web/chat/messages/skip-wait` | Ends the pause a `wait_start` status announced; the request carries on. Answers 202, `{"status": "wait_skipped", "turn": ...}` |
| `/api/v1/web/chat/messages/steer` | Also takes `content` and `file_ids`: stores the message and hands it to the running request, which reads it at its next step. Answers `{"status": "steered", "turn_id": ...}` |

`turn` says where XTM One reached the request: `local`, `remote` on another server of the platform, or `none` when nothing ran. Add `"awaiting_send": true` to a stop sent while your own message is on its way: answered `none`, the stop is kept for about 30 seconds, and that request ends `cancelled` as soon as it starts. A stop or a skip answers 503 when the request runs on another server and the message broker of XTM One cannot be reached: nothing reached it, so try again.

A steered message that no request takes answers 409: none runs, it was stopped, or it ended on the way. Nothing is stored and the message does not count against the limit: send it as a new message. An empty message answers 400, and too many messages 429 with `Retry-After`. `{"status": "stored", "turn_id": null}` means that the message is stored but no request is known to have taken it. With `"answered": false`, no request took it and no reply answers it. Either way, it stays in the conversation and shapes the next request: do not send it again. A 503 means nothing was stored: try again.

## Approve a tool call

An agent set to ask before some tools pauses on them when your message carried `"supports_tool_approval": true`. It sends `approval_required`, whose `proposals` each name a `tool_call_id`. Answer with `POST /api/v1/web/chat/messages/approve`:

```json
{"conversation_id": "9a7f2e1d-4c3b-4e5a-8f6d-1b2c3d4e5f60", "decisions": [{"tool_call_id": "call_5b1e0c7d", "decision": "approve"}]}
```

`decision` is `approve`, `approve_always` or `reject`, with an optional `rejection_reason`. XTM One answers `{"status": "accepted", "decided": 1}`, with the number of decisions it applied. It answers 400 without any decision, 404 for an unknown conversation, and 409 when nothing waits for an approval.

After a restart of your client, `GET /api/v1/web/chat/conversations/{conversation_id}/pending-approvals` returns what still waits, as `{"proposals": [...], "turn": "running"}`, or `"idle"`. It answers 503 when the message broker cannot say, rather than an `idle` that would drop a prompt the request still waits on.

## Mention several agents

To have several agents answer one message, add their ids in the order they answer:

```json
{"content": "@crisis-briefer @night-shift-triage what do we tell the crisis cell?", "mentioned_agent_ids": ["7c0e5d2a-1b7f-4a51-9c1e-2f4d8a6b3e90", "0f3b9a64-5d2c-4e8b-8a17-6c9e1d2f4a73"]}
```

The conversation's agent answers first, then each mentioned agent, in the same request: every frame carries the same `turn_id`. Each reply but the last arrives as a `done` with `"intermediate": true`, and a `status` frame with `"status": "mention_answering"`, `agent_name`, `position` and `total` announces the next agent. Every reply of such a request carries `answered_by`, with `agent_id`, `agent_name` and, when the agent has one, `agent_icon`; the General Assistant's own replies carry none. A note written in place of an agent that did not answer carries `"mention_failed": true`. Over REST, the replies after the first are in `mention_messages`.

A request can become such a request without your mentioning anyone: a message folded into it from elsewhere may mention agents. To tell your agent's reply from theirs, compare `answered_by.agent_id` with the conversation's agent.

## Reference other conversations

To have the agent read other conversations, add their ids, in the streamed message and over REST alike:

```json
{"content": "@weekly-velvet-harbor-brief what did we decide there?", "referenced_conversation_ids": ["3e8d1c52-7a49-4f06-b2d1-9c5e0a7f6b18"]}
```

`GET /api/v1/chat/conversation-references` lists the conversations you can reference. Every word of `q`, separated by spaces, hyphens or underscores, must appear in the title; `limit`, from 1 to 20, defaults to 8, and `exclude` leaves one conversation out. Each entry has `id`, `title`, `key` (what the interface writes after `@`), `updated_at`, `is_own` and, for a conversation shared with you, `owner_name`.

XTM One keeps at most 5 references, only to conversations you can open, never the conversation the message is sent in. The stored user message lists them in `conversation_refs`, with `conversation_id`, `title` and `key`.

## Requests you did not send

A conversation can be open in several places: another tab, another device, voice mode, a script, a background task that reports back. To see what the others do as it happens, follow the conversation:

```text
GET /api/v1/web/chat/conversations/{conversation_id}/events?client_id=soc-siem-script
```

This stream receives every request of the conversation that your client did not send, so that each place shows the answer as it is written. It also receives `user_message` frames, a user message stored from another place in `message`, and `conversation_title` and `approval_resolved` frames. It ends after 15 minutes, or when your client reads nothing for about 10 seconds: open it again. Two fields tell the frames apart:

| Field | Where | Meaning |
| --- | --- | --- |
| `relayed` | Every frame of a request your client did not send, and `user_message` frames | `true`: the request was sent elsewhere. Frames of your own requests never carry it |
| `turn_id` | Every frame of a request, from `turn_started` or `folded_into_active_turn` to its end | Names the request. Your frames and the copies other clients receive carry the same value. `user_message`, `conversation_title` and `session_expired` carry none |

An events stream opened while a request runs first receives a `thinking` frame with `"resumed": true`. Then come the steps, the reasoning and the answer written so far, then the rest as it comes. A request your client sent is not repeated when it opens the stream again with the same `client_id`. A script that cares only about its own messages ignores every frame with `relayed`, except those of the request its message was folded into.

## Messages folded into a running request

A conversation runs one request at a time. A message sent while another request runs is folded into it: that request takes your message into account, and you receive:

```json
{"type": "folded_into_active_turn", "conversation_id": "9a7f2e1d-4c3b-4e5a-8f6d-1b2c3d4e5f60", "folded": true, "turn_id": "c41f0e8a2b7d4c96a3e15f8d0b7c2e49"}
```

The frames of that request then follow on the same response, with `relayed` and that `turn_id`, up to its end. The agents your message mentions answer in that request, after its own agents, or in the next one when it ends first. Rarely, the message waits for the next request instead: `"folded": false`, with a `turn_id` only when a request is running, which picks the message up. Over REST, the same cases answer HTTP 202 with `"status": "folded_into_active_turn"` or `"status": "queued_for_next_turn"`.

## How a request ends

Every request you are shown ends with a `done`, `cancelled` or `error` frame, also one sent elsewhere, and also when the message broker of XTM One was briefly lost or the process that ran the request stopped. When XTM One ends a request itself, the `error` frame carries a `reason`:

| `reason` | Meaning |
| --- | --- |
| `interrupted` | The request stopped before its answer, during a restart of the server for example |
| `lost` | The request is over, but how it ended could not be delivered. Read the conversation to see what it stored |

A request still running when a restart stops waiting for it stores an assistant message before that `error`: it says the request was interrupted and names the tool calls that completed, those still running and the last action completed. Read back, that message has `"interrupted": true` and the completed calls in `tool_call_trace`. The REST request that started it answers 503 with `Retry-After`: read the conversation before you send the message again.

When the restart was the platform's own, a rolling update or a recycled worker, XTM One resumes the request by itself, unless you sent another message or stopped it since. The events streams of the conversation then receive a `turn_resuming` frame, followed by the frames of the resumed request.

A request that ended before storing its answer, or whose agent failed, leaves its tool work to the next message of the conversation, for 24 hours: the agent receives the calls it made and their results, and continues instead of calling the same tools again.

On the stream that runs it, a stopped request can end with two `cancelled` frames: the acknowledgement, then, once the stop is stored, a second one with `message_id`. In the same way, a failed agent can end with an `error` without `reason`, then a `done` with `message_id` once its reply is stored. Either frame means the request is over; the second says what to read back.

If your script waits for an answer that does not come, read `GET /api/v1/chat/conversations/{conversation_id}`: `is_processing` is `false` once no request runs, and the messages hold the answer. XTM One guarantees the end frame; this check covers a network that drops frames on your side.

## REST routes

| Route | What it does |
| --- | --- |
| `POST /api/v1/chat/conversations` | Creates a conversation, answers 201. Body: `agent_id`, the agent that answers, `title`, `workspace_id` |
| `GET /api/v1/chat/conversations` | Your conversations |
| `GET /api/v1/chat/conversations/{conversation_id}` | A conversation with its latest messages (`limit`) and `is_processing` |
| `GET /api/v1/chat/conversations/{conversation_id}/messages` | Older messages, before the message `before` |
| `POST /api/v1/chat/conversations/{conversation_id}/messages` | Sends a message and waits for the answer: `user_message`, `assistant_message`, `mention_messages`. The same fields as `POST /api/v1/web/chat/messages`, and it accepts a personal API key |
| `POST /api/v1/chat/conversations/{conversation_id}/upload` | Uploads a file. With `create_message=false`, it only stores the file and returns its `file_id` for the next message |
| `GET /api/v1/agents` | The agents you can use, with their `id` and `name` |

:::example[Ask the night shift triage agent from a SOC script]
Paul Garnier, SOC analyst on the night shift at Grand Meridian Hotels, wants the **Night shift triage** agent to summarize an alert from the script that already reads the SIEM. The script signs in with an account without two-factor authentication, kept in the environment as `XTM_ONE_URL`, `XTM_ONE_EMAIL` and `XTM_ONE_PASSWORD`, and prints the answer as it is written:

```python
import json, os

import requests

BASE = os.environ["XTM_ONE_URL"]  # https://xtm-one.grand-meridian.example
token = requests.post(f"{BASE}/api/v1/auth/login", json={
    "email": os.environ["XTM_ONE_EMAIL"],
    "password": os.environ["XTM_ONE_PASSWORD"],
}).json()["access_token"]
headers = {"Authorization": f"Bearer {token}", "X-Chat-Client-Id": "soc-siem-script"}

session = requests.post(f"{BASE}/api/v1/web/chat/sessions", headers=headers,
                        json={"agent_slug": "night-shift-triage"})
session.raise_for_status()
conversation_id = session.json()["conversation_id"]


def ask(question: str) -> str:
    with requests.post(f"{BASE}/api/v1/web/chat/messages", headers=headers, stream=True,
                       timeout=(10, 60),
                       json={"conversation_id": conversation_id, "content": question}) as response:
        response.raise_for_status()  # refused before the request started
        for line in response.iter_lines(decode_unicode=True):
            if not line.startswith("data: "):
                continue  # blank lines and ": keepalive"
            frame = json.loads(line[len("data: "):])
            if frame["type"] == "stream":
                print(frame["content"], end="", flush=True)
            elif frame["type"] == "done" and not frame.get("intermediate"):
                return frame["message_id"]
            elif frame["type"] in ("error", "cancelled") and not frame.get("provisional"):
                raise RuntimeError(frame.get("content") or frame["type"])
    raise RuntimeError("The stream ended without an answer: read the conversation")


ask("PowerShell started from Outlook on FD-PAR-03 at 23:12. "
    "Is it linked to Velvet Harbor, and what should I check first?")
```

The read timeout of 60 seconds is well above the 15 seconds between keepalives. The script keeps the frames flagged `relayed`: on its own response, they belong to the request its message was folded into. The agent reads OpenCTI and OpenAEV as Paul, and the answer prints piece by piece: the loader of Velvet Harbor's January intrusion started the same way, with the checks to run on the workstation.
:::

## What's next

- [Ask an agent in the chat](/xtm-one/use/chat/), the same conversation in the interface.
- [MCP servers](/develop/apis/mcp/), to give your AI client the tools of the suite.
- [XTM One emergency stop API](/develop/apis/emergency-stop/), to stop every agent at once.
