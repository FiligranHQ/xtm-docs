---
title: Try email and SMS injects on yourself before a simulation
description: Send an email or SMS inject of a scenario or simulation to yourself alone, read the result in the Tests tab, fix the content and replay the test until it is right.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/inject-tests/
---

A test sends an email or SMS inject to you, and to no one else, so that you can read what the players will receive before the simulation starts: the subject, the body, the variables, the links. The players see nothing, and the inject stays as it is in the scenario or simulation. This page tests one inject or several, reads the result, and replays the test after a fix. Injects in general are explained in [Add injects to scenarios and simulations](/openaev/use/injects/).

::diagram{id=openaev-injects-tests-flow}

## Before you start

- Your role has **Manage assessment**, or the right to manage the scenario or simulation. **Replay all tests** also needs **Launch assessment**, or the right to launch it.
- The inject sends an email or an SMS. Only those two can be tested: other injects have no **Test** action, and a bulk test skips them.
- Your account has an email address, and a phone number for SMS: the test goes to you.
- The platform can send email and SMS. See [Built-in injectors](/openaev/use/injectors/built-in/).

## Test one inject

:::steps

1. Open the scenario or the simulation, then its **Injects** tab.
2. Open the menu at the end of the inject's line, click **Test**, then confirm. The **Test** action is missing on injects that are neither email nor SMS.
3. A message confirms that the test was sent. Click **its dedicated page** in it to open the result.
4. Read your mailbox or your phone: the message is the one the players will get.

:::

## Test several injects at once

:::steps

1. In the **Injects** tab, tick the injects to test, or the box of the header to select them all.
2. In the toolbar that appears, click the **Test** icon, then confirm. Only email and SMS injects are tested; the dialog says so.
3. Click **its dedicated page** in the confirmation to open the **Tests** tab.

:::

## Read the result

The **Tests** tab of the scenario or simulation lists one line per tested inject, with its **Inject title**, its **Test execution time** and its **Test status**. Click a line to open **Test Details**: the traces of the sending, such as the delivery of the email to the mail server, or the error that stopped it. Until you test a first inject, the tab reads **No test yet**.

::screenshot{id=openaev-injects-test-list alt="The Tests tab of the Velvet Harbor replay scenario: the email inject Fake summit booking request, tested on April 12, 2026 at 10:47 AM, with the status EXECUTED."}

::screenshot{id=openaev-injects-test-detail alt="The Test Details of the Fake summit booking request: its start and end dates, a duration of 0 seconds, and the trace EXECUTED Mail sent to docs-admin@example.com."}

Each inject keeps its latest test only: a new test of the same inject replaces the previous one.

| Test status | What it means |
| --- | --- |
| **EXECUTED** | The platform sent the message. Check that it arrived and reads well |
| **ERROR** | The sending failed. Open the test to read why, often the mail or SMS configuration |

## Replay a test

- To test one inject again after a fix, open the menu at the end of its line in the **Tests** tab and click **Replay test**.
- To test them all again, click **Replay all tests** at the top of the list.
- To clear a result, click **Delete test** in the line's menu. The inject is not affected.

:::example[Proofread the fake summit booking request]
The **Velvet Harbor replay** scenario of Grand Meridian Hotels opens with an email inject, **Fake summit booking request**, sent to the teams **Front desk France** and **Front desk Spain**: the booking request for 12 rooms during the European Hospitality Summit 2026, under the subject `Group booking: European Hospitality Summit 2026`.

Before the replay, the exercise planner opens the scenario, the **Injects** tab, and clicks **Test** in the inject's menu. The **Tests** tab reads **EXECUTED**, and **Test Details** shows `Mail sent to` the planner's own address. In their mailbox, the planner finds that the body still gives last year's summit dates. They fix the body in the inject, click **Replay test**, and check the new email before the simulation starts.
:::

## What's next

- [Send a phishing exercise to your teams](/openaev/use/phishing/run/): phishing injects, with their landing page.
- [Run a simulation](/openaev/use/simulations/).
- [Inject execution statuses](/openaev/use/injects/status/): the statuses of the traces.
