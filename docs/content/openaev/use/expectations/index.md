---
title: How expectations score an inject
description: What each expectation type measures, who answers it, how OpenAEV combines the answers of agents, assets, persons and groups into a score, and when an unanswered expectation expires.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, exercise-planner, soc-analyst]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/expectations/overview/
---

An expectation says what must happen for an inject to count as a success: that Meridian EDR prevents the command, that the SOC detects it, that the night auditor reports the phishing email. OpenAEV creates one per target when the inject runs, collects the answers of your security tools and of your people, and turns them into the scores on the result of the inject. This page explains the types, who answers each one, how the answers add up, and when an expectation expires. To add expectations to an inject, see [Set the expectations of an inject](/openaev/use/expectations/management/); to answer them yourself, [Add expectation results by hand](/openaev/use/expectations/validation/).

## The types and who answers them

| Type | What it measures | Who answers |
| --- | --- | --- |
| **Prevention** | A security control blocked the action | Collectors of your security platforms, such as an EDR or an email security platform |
| **Detection** | A security tool raised an alert or an incident | Collectors of your security platforms, such as an EDR or a SIEM |
| **Vulnerability** | The target carries a vulnerability | The action itself, such as a vulnerability scanner, through its output |
| **Manual** | A person did what the exercise expects: decided, escalated, reported | An organizer of the exercise |
| **Article** | The targeted persons read the media pressure articles | The platform, when a person reads the article |
| **Challenge** | The targeted persons solved the challenges | The platform, when a person submits a right answer |

Prevention, detection and vulnerability are technical: they apply to assets and their agents. Manual, article and challenge measure a human response: they apply to persons and their teams. The actions of the Threat Arsenal say which types an inject can carry, and which it gets by default.

Each expectation can name the **Expected security platforms** that must answer it: EDR, XDR, SIEM, SOAR, NDR, EMAIL SECURITY and others. With none, any connected security platform may answer.

## The results you read

| Type | Success | Failure | In between | Not answered yet |
| --- | --- | --- | --- | --- |
| Prevention | **Prevented** | **Not Prevented** | **Partially Prevented** | **Pending** |
| Detection | **Detected** | **Not Detected** | **Partially Detected** | **Pending** |
| Vulnerability | **Not vulnerable** | **Vulnerable** | **Partially Vulnerable** | **Pending** |
| Manual, article, challenge | **Success** | **Failed** | **Partial** | **Pending** |

A result is a score from 0 to 100. Each expectation has a success score: a result at or above it is a success, below it a failure. The success score is 100 by default, and 50 for a manual expectation (`openaev.expectation.manual.default-score-value`).

## How the answers add up

::diagram{id=openaev-expectations-scoring}

### On one agent or one person

An agent, or an asset without an agent, can receive several answers: one per security platform, plus the ones you add by hand.

- **Prevention and detection**: the expectation is settled as soon as one platform reaches the success score, without waiting for the others. A later answer is recorded but cannot lower it: a negative result never overrides a positive one. Without a success, OpenAEV waits for every expected platform, then keeps the highest score.
- **Vulnerability**: OpenAEV waits for every source. A **Vulnerable** answer wins over the others, even when the other sources found nothing.
- **Manual, article and challenge**: a new answer replaces the previous one.

### On an asset, a group or a team

The scores then go up: from the agents to their asset, from the assets to their asset group, from the persons to their team. At each level, the **Validation mode** of the expectation decides:

| Validation mode | Success | Failure | Otherwise |
| --- | --- | --- | --- |
| **All assets (per group) must validate the expectation**, or **All persons (per team)** | Every member succeeded | One member failed | **Pending** |
| **At least one asset (per group) must validate the expectation**, or **At least one person (per team)** | One member succeeded | Every member failed | **Pending** |

A success gives the level the success score, a failure gives it 0. An asset targeted on its own, or a person, counts as a group of one. "All" suits a baseline that every workstation must meet; "at least one" suits a detection that one alert of the SOC is enough to prove.

In a simulation, only the enabled persons of a team get an expectation, and a team without any gets none.

### When no security platform can answer

Only collectors of security platforms answer prevention and detection. When the inject runs, OpenAEV looks for a collector of one of the expected platform types, or of any type when none is set. If there is none, it fails the expectation at once, with a score of 0, instead of letting it wait. Its card reads **No automated security platform detected**. The check happens once, when the inject runs; vulnerability expectations are not concerned.

## When an expectation expires

An expectation waits for its answers for a set time, counted from the run of the inject. When the time is up, every target still without an answer gets one from the expiration manager:

- prevention and detection fail: **Not Prevented**, **Not Detected**;
- manual, article and challenge fail: **Failed**;
- vulnerability succeeds: **Not vulnerable**. No scanner reported anything, so nothing was found.

| Expectation | Default | Configuration key, in seconds |
| --- | --- | --- |
| Prevention, detection, vulnerability | 6 hours | `openaev.expectation.technical.expiration-time` (21600), then `openaev.expectation.prevention.expiration-time`, `openaev.expectation.detection.expiration-time`, `openaev.expectation.vulnerability.expiration-time` for one type |
| Manual, article, challenge | 24 hours | `openaev.expectation.human.expiration-time` (86400), then `openaev.expectation.manual.expiration-time`, `openaev.expectation.article.expiration-time`, `openaev.expectation.challenge.expiration-time` for one type |

A key for one type wins over the key of its family. Each expectation can also set its own **Expiration time** in the inject. OpenAEV never lets a prevention or a detection expire before its collectors have polled twice: it extends the time to twice the longest polling period of the expected collectors. The keys are in the [configuration reference](/reference/openaev/configuration/#openaev-expectation).

A simulation finishes once every enabled inject has run and every expectation of the injects that did not fail is answered or expired.

:::example[Score the PowerShell loader on the front-desk group]
The `PowerShell loader` inject of Grand Meridian Hotels runs on the asset group **Front desk workstations**, five endpoints with one agent each. It expects a **Prevention** with **At least one asset (per group) must validate the expectation**, and a **Detection** with **All assets (per group) must validate the expectation**, both from an **EDR**, success score 100.

The Nice workstation is off that night: its agent is inactive, so it gets no expectation. Meridian EDR's collector reports a block on `fd-madrid-02` and alerts on Madrid and Paris; Barcelona sends nothing. The group's prevention is **Prevented** at once: one asset is enough. Its detection stays **Pending** until the 6 hours run out, then Barcelona expires as **Not Detected**, and the group reads **Not Detected**. The SOC manager learns that one front desk raised no alert at all.
:::

## What's next

- [Set the expectations of an inject](/openaev/use/expectations/management/).
- [Add expectation results by hand](/openaev/use/expectations/validation/).
- [Read the result of an inject](/openaev/use/injects/results/).
- [Collectors](/openaev/use/collectors/): connect the security platforms that answer.
