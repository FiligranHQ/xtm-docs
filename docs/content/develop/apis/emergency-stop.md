---
title: XTM One emergency stop API
description: The REST routes of the XTM One emergency stop - press it, record why, lift it, read its state, manage the emergency token - with their fields, answers and errors.
doc_type: reference
products:
  - xtm-one
personas: [developer, platform-engineer, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
no_screenshot: "This page describes HTTP routes: the screens that press the stop and create the token are on the Stop every agent at once page."
migrated_from:
  - xtm-one:/administration/emergency-stop-api/
---

Everything the [emergency stop](/xtm-one/administer/emergency-stop/) does in the XTM One interface is available over REST, plus one route that needs no session: the one a printed runbook or a monitoring hook uses to stop the whole platform. This page is the reference of the routes under `/api/v1/panic`: who may call each one, its fields, its answer and its errors. None of them depends on the edition or on the AI switch: the stop is what switches AI off, so it keeps answering while AI is off.

::diagram{id=develop-apis-emergency-stop}

In the examples, the platform is `https://xtm-one.grand-meridian.example`. When a reverse proxy serves XTM One under a path (`BASE_PATH`), that path is part of the address, as in `https://intranet.example.com/xtm-one/api/v1/panic/emergency`; the command shown in :menu[Administration > Settings > Access and security] and on the printed procedure already includes it.

## Authentication

| Credential | Header | Accepted by | Allows |
| --- | --- | --- | --- |
| A session token, or a personal API key `fcp-...` created in the profile ([Set up your profile and API keys](/xtm-one/use/profile/)) | `Authorization: Bearer fcp-...` | Every route except `POST /api/v1/panic/emergency` | What the user may do: their own scope for every user; the platform scope and the token routes for administrators |
| The emergency stop token `xes-...`, generated in :menu[Administration > Settings > Access and security], section **Emergency stop** | `Authorization: Bearer xes-...` | `POST /api/v1/panic/emergency` only | Stop the whole platform, nothing else |

## Routes

| Method and path | Who | Purpose |
| --- | --- | --- |
| `POST /api/v1/panic/stop` | Any user; scope `platform` needs an administrator | Press the stop |
| `POST /api/v1/panic/stop/{audit_id}/reason` | The user who pressed that stop | Record why, after the stop |
| `POST /api/v1/panic/resume` | Any user; scope `platform` needs an administrator | Lift the stop, with a reason |
| `GET /api/v1/panic/status` | Any user | Whether a stop is in effect, what a resume would re-enable, who pressed the last stop |
| `GET /api/v1/panic/token` | Administrators | Whether an emergency token exists, never the token itself |
| `POST /api/v1/panic/token` | Administrators | Generate or rotate the emergency token, returned once |
| `DELETE /api/v1/panic/token` | Administrators | Revoke the emergency token |
| `POST /api/v1/panic/emergency` | The emergency token | Stop the whole platform without a session |

## Press the stop

```bash
curl -X POST https://xtm-one.grand-meridian.example/api/v1/panic/stop \
  -H "Authorization: Bearer ${XTM_ONE_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"scope": "platform"}'
```

| Field | Values | Meaning |
| --- | --- | --- |
| `scope` | `self` (default), `platform` | `self` stops everything that runs for you. `platform` stops every user's activity and the runs that belong to no user, and switches AI off; it answers `403` to a user who is not an administrator |

The interface asks for a typed code before it calls this route; the route itself does not: it is a safeguard of the interface, not a check of the server. The stop runs at once and the answer says what it did:

```json
{
  "scope": "platform",
  "runs_cancelled": 3,
  "executions_cancelled": 4,
  "sub_agent_tasks_cancelled": 1,
  "chat_turns_cancelled": 0,
  "flow_runs_cancelled": 0,
  "runner_commands_cancelled": 0,
  "evaluation_runs_cancelled": 0,
  "autonomous_runs_cancelled": 0,
  "assignments_paused": 12,
  "ai_disabled": true,
  "late_admissions_cancelled": 0,
  "completed": true,
  "audit_id": "5f0c6d0a-2c8f-4a4e-9a35-1a6c9d2e7b10",
  "stopped_at": "2026-04-12T00:43:04.628632+00:00"
}
```

| Field | Meaning |
| --- | --- |
| `runs_cancelled` | Runs stopped: in progress, queued or waiting for a person, the runs they handed work to, and any run admitted while the stop was being applied |
| `executions_cancelled` | Background executions released, so that no worker resumes them |
| `sub_agent_tasks_cancelled` | Background tasks delegated to other agents |
| `chat_turns_cancelled` | Chat answers being written that were asked to stop |
| `flow_runs_cancelled` | Flow runs in progress |
| `runner_commands_cancelled` | Commands for a computer not yet dispatched to it. One already dispatched cannot be called back, but it is never sent again |
| `evaluation_runs_cancelled`, `autonomous_runs_cancelled` | Runs that belong to no user, such as the autonomous attack paths driven for OpenAEV. Scope `platform` only |
| `assignments_paused` | Assignments whose schedules and triggers no longer fire |
| `ai_disabled` | `true` when the stop switched AI off (scope `platform`) |
| `late_admissions_cancelled` | Runs another server admitted while the stop was being applied, cancelled by the follow-up pass. Already counted in `runs_cancelled`, reported apart so you know a second wave happened |
| `completed` | `false` when cancelling the running work failed after the assignments were paused (and AI switched off, for the platform). The follow-up passes cancelled what they reach, and the counts are what was really halted. Read the server log and press the stop again if work still runs |
| `audit_id` | The log entry of this stop: pass it to record the reason |
| `stopped_at` | When the stop took effect |

Pressing the stop twice is safe: the second press finds nothing left to cancel and answers zeros.

## Record why

The stop never waits for a reason. Once it has run, attach the reason to the entry it returned:

```bash
curl -X POST https://xtm-one.grand-meridian.example/api/v1/panic/stop/5f0c6d0a-2c8f-4a4e-9a35-1a6c9d2e7b10/reason \
  -H "Authorization: Bearer ${XTM_ONE_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"reason": "Night shift triage creating Jira tickets in a loop"}'
```

```json
{"audit_id": "5f0c6d0a-2c8f-4a4e-9a35-1a6c9d2e7b10", "assignments_annotated": 12}
```

The reason, 1 to 2,000 characters, is written on the stop's log entry and on every assignment that stop paused, where the banner and the assignment's page show it. Only the user who pressed the stop may record it: anyone else gets `404`. A blank reason gets `422`.

## Lift the stop

```bash
curl -X POST https://xtm-one.grand-meridian.example/api/v1/panic/resume \
  -H "Authorization: Bearer ${XTM_ONE_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"scope": "platform", "reason": "Triage loop fixed in the agent instructions"}'
```

| Field | Values | Meaning |
| --- | --- | --- |
| `scope` | `self` (default), `platform` | The same rules as the stop |
| `reason` | Text, 1 to 2,000 characters, required | Recorded on the resume entry and on every assignment re-enabled |

```json
{"scope": "platform", "assignments_resumed": 12, "ai_enabled": true}
```

Only the assignments the emergency stop paused are re-enabled; one paused by hand stays paused. For the scope `platform`, the AI switch goes back to what it was before the stop (on, off, or following the deployment default), and only if it still reads as the stop left it: a change made in the settings during the stop is kept. `ai_enabled` is the state that results: on a platform started with AI off, or where AI was off before the stop, it stays `false`.

A `self` resume while a stop of the whole platform is in effect answers `409` with the code `platform_stop_active`: only an administrator lifting the platform stop resumes anything then.

## Read the state

```bash
curl https://xtm-one.grand-meridian.example/api/v1/panic/status \
  -H "Authorization: Bearer ${XTM_ONE_API_KEY}"
```

```json
{
  "self_paused_assignments": 0,
  "self_stop_active": false,
  "platform_paused_assignments": 12,
  "ai_disabled": true,
  "platform_stop_active": true,
  "active": true,
  "last_stop": {
    "at": "2026-04-12T00:43:04.628632+00:00",
    "by": "Hélène Marchetti",
    "scope": "platform",
    "reason": "Night shift triage creating Jira tickets in a loop"
  }
}
```

| Field | Meaning |
| --- | --- |
| `self_paused_assignments` | The assignments of your scope a `self` resume would re-enable. One that another person's stop still covers, such as an assignment you own that runs as them, is not counted: it comes back once their stop is lifted too |
| `self_stop_active` | Your own stop is in effect and still covers a paused assignment. Your resume stays available while it is `true`, even when it would re-enable nothing yet |
| `platform_paused_assignments` | The assignments a `platform` resume would re-enable. `null` for a user who is not an administrator |
| `ai_disabled` | AI is off because of a stop of the whole platform. AI switched off by hand in the settings is not reported here, and a resume does not touch it. `null` for a user who is not an administrator |
| `platform_stop_active` | A stop of the whole platform is in effect. Visible to every user |
| `active` | A stop is in effect for you: the platform's or your own |
| `last_stop` | The latest stop that concerns you: when, who (their display name), the scope, and the reason (`null` until it is recorded) |

The interface reads this route every 15 seconds to show the banner.

## Manage the emergency token

Administrators only. There is one token per platform.

| Call | Answer |
| --- | --- |
| `GET /api/v1/panic/token` | `configured`, and when it is: `prefix` (`xes-` and its first 8 characters), `created_at`, `created_by` (an email address), `last_used_at` |
| `POST /api/v1/panic/token` | Generates a token, or replaces the current one at once: `token` (`xes-` and 48 hexadecimal characters), returned this once, with `prefix`, `created_at` and `created_by`. Only a hash is stored: a lost token cannot be recovered, rotate it |
| `DELETE /api/v1/panic/token` | Revokes it: `204`, or `404` when none exists |

Generating, rotating and revoking are recorded in the logs.

## Stop without a session

The command the printed procedure carries:

```bash
curl -X POST https://xtm-one.grand-meridian.example/api/v1/panic/emergency \
  -H "Authorization: Bearer ${XTM_ONE_EMERGENCY_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"reason": "Runbook step 3: agent looping on the ticket queue"}'
```

| Field | Meaning |
| --- | --- |
| `reason` | Optional, up to 2,000 characters: a runbook usually states the cause, a monitoring hook may not. Recorded on the stop's log entry and on the paused assignments when given |

The answer is the one of `POST /api/v1/panic/stop`. The stop always covers the whole platform and is attributed to the administrator who generated the token. If that account is no longer an active administrator, it is attributed to the oldest active administrator; a valid token still works when no active administrator is left. The token's `last_used_at` is updated.

A missing or wrong token gets `401`, and the attempt is recorded in the logs as a security event, filed under the administrator who owns the token. The token is refused by every other route.

The route is limited per client address: 10 attempts per 600 seconds by default, counted before the token is checked, set by `PANIC_EMERGENCY_RATE_LIMIT` and `PANIC_EMERGENCY_RATE_WINDOW_SECONDS` (see the [XTM One configuration](/reference/xtm-one/configuration/#advanced-tuning)). Past the limit it answers `429` with a `Retry-After` header. If the limiter cannot be reached, it answers `503` rather than becoming unlimited. Neither refusal applies to a request carrying the prefix of the current token, so whoever spent the limit from the same address cannot lock the holder of the real token out.

## Errors

| Status | When |
| --- | --- |
| `401` | No credential or an invalid one; on `POST /api/v1/panic/emergency`, a missing or wrong emergency token |
| `403` | Scope `platform`, or a token route, called by a user who is not an administrator |
| `404` | Recording the reason of a stop that is not yours; revoking a token when none exists |
| `409` | Re-enabling through `PUT /api/v1/assignments/{id}` an assignment the emergency stop paused (`code: "emergency_stop"`): lift the stop instead. A `self` resume while a stop of the whole platform is in effect (`code: "platform_stop_active"`) |
| `422` | An unknown `scope`, or a missing or blank `reason` where one is required |
| `429` | Too many attempts on `POST /api/v1/panic/emergency` from one address: wait for `Retry-After` |
| `503` | The limiter of `POST /api/v1/panic/emergency` cannot be reached |

## What "at once" means

The stop is two steps, in this order. First it closes the door: every assignment in scope is paused and, for the platform, AI is switched off, in a transaction of its own, so every server sees a paused assignment and refused AI from then on. Then it cancels what is live. A run inside a model or tool call notices within about two seconds; a chat answer being written, within one second.

A scheduler on another server that read the previous state just before the door closed can still create one last run. A follow-up pass right after the stop cancels those and reports them in `late_admissions_cancelled`; a second pass a few seconds later, once no server can still believe AI is on, counts into the log entry only. For a `self` stop, that second pass cancels only the runs that existed before the stop, the runs of the assignments it paused and the runs started by a run it cancelled: what you start after your stop is yours.

## What's next

- [Stop every agent at once](/xtm-one/administer/emergency-stop/): the interface, the printed procedure, what users see.
- [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/)
- [XTM One configuration](/reference/xtm-one/configuration/)
