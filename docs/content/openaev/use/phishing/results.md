---
title: Read who opened, clicked and submitted
description: Read the results of a phishing exercise person by person and team by team, tell a person's click from a mail gateway's, and find the credentials it captured.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/build/components/phishing/results/
---

The results of a phishing exercise say, for every person who received the lure, whether they opened it, followed its link and submitted the form, with where and when each happened. They also say whether your email security caught the email. This page reads them, explains why green means resisted, and shows how to tell a person from a machine. To send an exercise first, see [Send a phishing exercise to your teams](/openaev/use/phishing/run/).

## Before you start

- Your role has **Access assessment** to read the simulation or the atomic test, and **Access findings** to read the **Findings** list.

## Read the results of the inject

:::steps

1. Open the simulation, its **Injects** tab, and the phishing inject. For an atomic test, open it from :menu[Atomic testings].
2. The targets list each team, and under it each person. Their icons show the state of each family of expectations: **Human Response** for the three phishing steps, **Detection** and **Prevention** for your email security.
3. Select a person. Their expectation cards open, in the order of the attack: **Email not opened**, **Link not clicked**, **Credentials not submitted**, then **Detection** and **Prevention**.
4. On a red step, read its **Outcome:**, the time it happened, and its **Origin:**: the address the request came from (its client in a tooltip), the delay after delivery, such as `3 minutes after delivery`, and a **Possibly automated** chip when it applies.
5. Select a team to read its own verdict on each step, derived from its people.

:::

::screenshot{id=openaev-phishing-person-results alt="The Persons of the phishing inject, Lucía Ortega and Paul Garnier red and the three others green, and the results of Paul Garnier: Email not opened, Link not clicked and Credentials not submitted failed, each with its outcome, its time and its origin 10.20.4.11, 2 or 3 minutes after delivery."}

::diagram{id=openaev-phishing-results-steps}

## Green means resisted

The three phishing steps are named after what you want to see, so the colors read the other way from a detection:

- **Green**: the person has not done it. The card's outcome reads **No phishing interaction detected**.
- **Red**: the person fell for that step. The outcome reads **Opened the phishing landing page** or **Submitted data on the phishing page**. An exercise run before the upgrade to 3.261118.0 may also show **Opened the phishing email**, from the time the tracking image counted as an open.

Every step is green from the moment the email leaves, and phishing steps never expire: a person who ignores the email keeps three green steps, and you can read the results at any time. Only what the person does on the landing page turns a step red: their first move on the page (a mouse movement, a touch, a key or a scroll) turns **Email not opened** and **Link not clicked** red together, and a submission all three. The tracking image of the email alone never does, since mail scanners load it too: it leaves **Email not opened** green, reading **Email image loaded, open not confirmed**, with the origin of the request. Each red step keeps the origin of the request that turned it red.

A team follows the validation rule of the expectation, shown as **Validation rule:** on its card:

| Validation rule | The team turns red |
| --- | --- |
| **All persons (per team) must validate the expectation**, the default | As soon as one person of the team falls for the step: **A team member fell for the phishing** |
| **At least one person (per team) must validate the expectation** | Only when every person falls for it. Until then it reads **At least one team member resisted** |

::screenshot{id=openaev-phishing-team-results alt="The results of the team Front desk France: its three phishing steps failed, under the validation rule All persons (per team) must validate the expectation, with the outcome A team member fell for the phishing."}

## Tell a person from a mail gateway

Mail gateways, link checkers and chat previews open links and images on their own, often within seconds of delivery. OpenAEV does not score them:

- Loading the tracking image or the landing page counts for nothing on its own. Only the person's first move on the landing page, at least 800 milliseconds after it shows, turns a step red.
- A hit within 10 seconds of delivery, or whose client is a known scanner (Proofpoint, Mimecast, Microsoft Safe Links, Barracuda and others), leaves the step green. The card reads **Automated scanner activity detected and ignored**, with its origin and a **Likely automated** chip. A submission from a scanner is not captured either.
- A browser driven by automation tools, such as Selenium or Puppeteer, is ignored the same way, even when it moves on the page.
- A tracking image loaded by anything else, a person's mail client or an image proxy such as Gmail's, leaves **Email not opened** green, reading **Email image loaded, open not confirmed**. Through an image proxy, its origin carries a **Possibly automated** chip.

::screenshot{id=openaev-phishing-unconfirmed-open alt="The results of Camille Roux, who opened the email but never moved on the landing page: Email not opened stays green, reading Email image loaded, open not confirmed, with its origin 10.20.4.12, and her two other steps green."}

The chips are advice: they never change a score. A person whose **Email not opened** stays green may still have read the email: the step only proves who went as far as the landing page.

## Detection and prevention

**Detection** and **Prevention** say whether your email security caught the lure. Only a security platform of the **Email Security** type can fill them, through its collector; other security tools are not asked. Without such a platform, they stay empty and count as not detected and not prevented when they expire. See [Collectors](/openaev/use/collectors/).

A phishing exercise does not record who reported the email to your security team: count the reports your SOC received during the exercise.

## Find the captured credentials

When the landing page has **Capture submitted data** on, each first submission of a person creates a **Credentials** finding:

| Field | Value |
| --- | --- |
| Name | The name of the landing page |
| Value | The user name, followed by ` / ` and the password when **Capture passwords** is on. A form whose field names OpenAEV does not recognize gives every filled field as `name=value` pairs. The lists show it masked as a whole, user name included: `night.audit` reads `ni******` |
| Linked to | The person, their team and the inject |

::screenshot{id=openaev-phishing-credentials-finding alt="The Findings tab of the phishing inject: one Credentials finding, its value masked as ni******, first and last seen on April 12, 2026."}

They are in the **Findings** tab of the simulation and in :menu[Findings]. A credentials finding can feed a following inject: see [Chain and transfer injects](/openaev/use/injects/chaining/). Without capture, a submission turns its step red and stores nothing.

:::example[What the front desks did with the fake summit booking]
On 12 April 2026, a few minutes after the emails left, the awareness manager of Grand Meridian Hotels opens the inject **Fake EHS 2026 group booking** of the simulation **Front-desk phishing**, sent to the 5 people of `Front desk France` and `Front desk Spain`.

- 2 people have **Email not opened** and **Link not clicked** red, and 1 has **Credentials not submitted** red. Camille Roux and Julien Bernard loaded the email's images but never went to the page: their **Email not opened** stays green, reading **Email image loaded, open not confirmed**. Javier Morales never opened the email: his three steps stay green.
- Under the default rule, **Front desk France** is red on all three steps, and **Front desk Spain** on the first two: Lucía Ortega opened the page but submitted nothing.
- The one who submitted is Paul Garnier, the night auditor of the Paris hotel: his **Credentials not submitted** reads `3 minutes after delivery`, from `10.20.4.11`, the address of the front-desk workstation `fd-paris-01`. Its **Credentials** finding holds the shared account `night.audit`, without the password, because the page does not keep passwords.

Hélène Marchetti, the CISO, takes two actions: move the shared `night.audit` account to personal accounts, and connect the Meridian email gateway to OpenAEV, so that the next exercise also measures whether it caught the lure.
:::

## What's next

- [Expectations](/openaev/use/expectations/): how expectations, scores and validation rules work.
- [Findings](/openaev/use/findings/): the captured credentials, with the other findings.
- [Send a phishing exercise to your teams](/openaev/use/phishing/run/) again, after the awareness session.
