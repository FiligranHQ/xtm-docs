---
title: Review who changed what on the platform
description: Read the audit log of XTM One in Administration > Audit logs, find who created, changed, shared or deleted an agent or a setting, and keep the evidence of it.
doc_type: how-to
products: [xtm-one]
personas: [admin, auditor]
owner: docs-xtm-one
last_reviewed: 2026-11-16
---

XTM One writes an entry in its audit log each time someone signs in, creates, changes, shares or deletes something, and each time the platform acts on its own. :menu[Administration > Audit logs] reads that log: who did what, to which agent, assignment, integration or setting, when and from which address. Open it when something changed and nobody knows why, to answer an auditor, or to check a person's actions before you change their access. What agents did with the AI, step by step, is in their traces instead: see [Follow what agents do and what they cost](/xtm-one/administer/observability/).

::screenshot{id=xtm-one-audit-logs-list alt="The Audit logs table filtered on Sophie Laurent: the search, the Action, Entity, User and Time range filters and the count of her entries out of all on one row, then one row per entry with its date, its summary, its action and entity badges and her email, for the agents, assignments, custom tool, variables, skill and prompt she created and changed."}

## Before you start

- Your account is an administrator. **Audit logs** is in the **Administration** menu, which other users do not see, and the audit log's API answers administrators only.
- A user who is not an administrator reads the entries of their own actions only, through an agent that carries the **Platform Insights** tool. See [Get answers about the platform's own activity](/xtm-one/use/platform-insights/).

## What the log records

Each entry has a one-line summary ("Created agent 'Night shift triage'"), an **action**, the **entity** it touched with its name and identifier, the email of who acted, the IP address the request came from, and when. Some entries also keep details, such as the values a change set or the fields it touched.

The table and its filters show each action and entity by its name (**Sign-in**, **User group**); the event's JSON and the API keep the key the platform records (`login`, `group`), given in brackets below.

| Kind of event | Examples of actions | Entities |
| --- | --- | --- |
| Sign-ins | **Sign-in** (`login`), **Sign-in refused** (`login_refused`: a refused single sign-on, a disabled account) | **User** |
| Changes to what people build | **Create**, **Update**, **Delete**, **Share**, **Unshare**, **Toggle**, **Run now** (`trigger`), **Import**, **Export** | **Agent**, **Assignment**, **Integration**, **Knowledge Base**, **MCP Server**, **Custom Tool**, **Template**, **Presentation**, **Flow** |
| Administration | **Create**, **Update**, **Delete**, **Bind** and **Unbind** (a group member or manager), **Impersonate** | **User**, **User group**, **Password**, **API Key**, **SSO**, **LLM Provider**, **General Assistant**, **Channel Bot**, **OAuth Default**, **Security**, **Security Audit finding** |
| Settings | **Configure**, **Toggle**, **Create**, **Update** | **Observability**, **Platform settings**, **General Assistant** (which also holds the quotas), the security options |
| Security events | **Security alert** (`security_alert`: a prompt injection detected in a chat or a channel) | **Security** |
| Oversight | **Break glass** (`break_glass`: an administrator read another person's delegations or runner commands), **Inspect traces** and **Inspect audit log** (an agent read traces or this log for someone) | **Agent delegations**, **Runner commands** |

- **What the platform does on its own** is recorded too: an assignment paused automatically, the emergency stop. The email then starts with `system:` and names the component that acted; the entry is filed under the person it happened to.
- **Impersonation**: what an administrator does while signed in as someone else is recorded as the administrator's.
- **A read is recorded only when it is oversight**: an administrator reading another person's delegations or machine commands, or an agent reading traces or the audit log. Each is recorded once per person, object and 15 minutes. Opening a page, a conversation or this log in the browser is not recorded.
- **What agents say and do** during a run is not in this log: it is in the run's trace.

## Find an entry

Open :menu[Administration > Audit logs]. The log is a table, one row per entry, with the columns **Date**, **Summary**, **Action**, **Entity** and **User**, the newest entries first. The toolbar shows the number of entries; once a search or a filter narrows the list, it reads "12 of 480": the entries that match out of the whole log. The **i** next to the title, **About audit logs**, reminds you how many days entries are kept.

:::steps
1. Type in the search field (**Search...**) to match the summary, the entity's name or the email of who acted.
2. Click **Action**, **Entity** or **User** and select one value or several: the list keeps the entries that match one of them. Each button shows how many values it holds. The actions and entities offered are the ones the log holds.
3. Click **Time range** and choose **Last hour**, **Last 24 hours**, **Last 7 days**, **Last 30 days**, **Last 90 days**, or a **From** and **To** day on the line under it. **All time** removes the bound.
4. Click the header of **Date**, **Action**, **Entity** or **User** to sort on it; click it again to reverse the order.
5. Click a row to open **Audit Log Event**: the summary, who acted (their email opens their user page), then **Raw Event (JSON)** with every field and the details. **Copy** puts the JSON in your clipboard.
:::

**Clear all filters** removes the search, the filters and the time range at once. **Columns** hides **Action**, **Entity** or **User**, and shows **IP address**, hidden by default. The page size, 25 rows by default, changes from 10 to 100 rows per page, with the pager under the table. The search, the filters, the time range, the sort and the page stay in the address: bookmark a view ("every deletion of an agent in the last 30 days") or send it to another administrator.

A user's own page, tab **Activity**, shows the same table restricted to that person's entries, without the **User** column and filter (see [How access works in XTM One](/xtm-one/concepts/access/)).

## Keep the evidence

- **How long entries stay**: XTM One deletes the entries older than **Audit log retention (days)**, 90 by default, every hour. Each user's most recent sign-in stays past that window, so the users list still shows when they last signed in.
- **Keep them longer**: in :menu[Administration > Settings], tab **Usage and quotas**, section **Observability**, click **Configure**, set **Audit log retention (days)** from 30 to 3650 (ten years), and click **Save**. An empty field goes back to 90 days. Choose the window your audit policy requires before entries older than it are gone: a deleted entry does not come back. Saving writes the entry "Updated observability settings", with the old and the new window in its details. While AI is turned off or not licensed, the section shows the value but cannot be changed.
- **People who leave**: disable their account rather than delete it (see [How access works in XTM One](/xtm-one/concepts/access/)), and export their entries before any deletion you must make.
- **Export**: the page has no export button. Copy an event's JSON for one entry, or read the log from `GET /api/v1/audit-logs` with an administrator's API key: the same filters as the page (`search`, `user_id`, `action`, `entity_type`, each with one value or several separated by commas, and `date_from`, `date_to` in ISO 8601), the same sort (`sort` on `time`, `action`, `entity` or `user`, `direction` `desc` by default or `asc`), up to 200 entries a call with `limit` and `offset`. A scheduled script that loads it into your SIEM keeps the log longer than the retention window.

:::example[Find who raised the night shift's quota]
On Tuesday 14 April 2026, Julien Bernard, SOC manager of Grand Meridian Hotels, sees in **Model usage** that his team ran 340 traces on Sunday night, where the quota of 50 agentic executions a day per user should have stopped them. He tells Hélène Marchetti, who opens :menu[Administration > Audit logs] and sets:

| Filter | Value |
| --- | --- |
| Search | `SOC` |
| **Action** | **Update** |
| **Time range** | **Last 7 days** |

One entry remains: "Updated group 'SOC'", by marc.dupuis@grand-meridian.example on 10 April at 18:12. Its **Audit Log Event** gives the address it came from, 198.51.100.24. Marc confirms he set the group to **Unlimited** on its **Quotas** tab for the Velvet Harbor alert weekend. Hélène sets the limit back to 50 and notes the change in the crisis cell's log. Her own change shows at the top of the list right away, under her email.
:::

## What's next

- [Follow what agents do and what they cost](/xtm-one/administer/observability/), for what agents did during their runs.
- [How access works in XTM One](/xtm-one/concepts/access/), to disable, impersonate or delete an account.
- [Get answers about the platform's own activity](/xtm-one/use/platform-insights/), to ask "who deleted an agent last week?" in the chat.
