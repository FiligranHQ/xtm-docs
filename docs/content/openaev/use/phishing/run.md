---
title: Send a phishing exercise to your teams
description: Add a landing page's phishing action to a simulation, choose the email and the teams, try it on yourself with an atomic test, and check that every email left.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/build/components/phishing/run-a-campaign/
---

A phishing exercise goes out as an inject: the action a landing page created in the Threat Arsenal, with an email template and the teams that receive it. Every person of those teams gets their own email at the time the inject is scheduled. This page adds the inject to a scenario or simulation, tries it first on a test team, and checks the sending. You need a [landing page](/openaev/use/phishing/landing-pages/) and an [email template](/openaev/use/phishing/email-templates/) first.

## Before you start

- Your role has **Manage assessment** to add the inject, and **Launch assessment** to start the simulation or the atomic test.
- The teams exist and hold the people to reach, each with an email address. See [People](/openaev/use/targets/people/).
- The platform can send email, and its address, or the custom domain of the landing page, opens from where your people read their email. See [How a phishing exercise works](/openaev/use/phishing/#before-the-first-exercise).

## Add the phishing inject

:::steps

1. Open the scenario or the simulation, then its **Injects** tab, and click **Create**. The Threat Arsenal picker opens.
2. Search for **Phishing:** and the name of your landing page, then click its card. Every landing page has one action, named after it.
3. Under **General**, give the inject a **Title**, and under **Scheduling**, the delay after the start of the simulation when the emails go out.
4. Under **Targeted teams**, click **Modify target teams** and choose the teams, or turn on **All teams**. Each person of each team gets one email.
5. Under **Inject data**, choose the **Email template**. The first template is preselected.
6. To change the email for this inject only, fill in **Subject override**, **Sender name override** or **Sender email override**. They take precedence over the template.
7. Under **Inject expectations**, check the five expectations: the three steps **Email not opened**, **Link not clicked** and **Credentials not submitted**, then **Detection** and **Prevention**. Remove **Detection** and **Prevention** if no security platform of the **Email Security** type reports to OpenAEV: left empty, they count as not detected and not prevented when they expire.
8. Click **Create**.

:::

::diagram{id=openaev-phishing-run-delivery}

The inject goes out with the simulation, like any other: see [Run a simulation](/openaev/use/simulations/). A phishing inject cannot be tested with the **Test** action of the injects list, which covers email and SMS injects only: try it with an atomic test instead.

## Try it on yourself first

An atomic test runs the whole chain, from the sending to the score, without waiting for a simulation.

:::steps

1. Create a team that holds only you, or a few colleagues who know about the exercise.
2. Open the landing page, from :menu[Components > Phishing], and click **Create atomic test**. The atomic test opens on the phishing action.
3. Choose the email template and your test team, then create the atomic test.
4. Click **Launch now**, then **Confirm**.
5. Open the email you received, click its link, and submit the form.

:::

Each step of your result turns red in turn. Clicks within 10 seconds of the email are ignored as a mail gateway's inspection: wait a little before you click. Delete the atomic test when you are done: its finding would otherwise stay in the **Findings** list.

## Check that every email left

Open the inject, then **Execution details**. The traces end with **Phishing emails sent to** followed by the number of people, counting every person of the targeted teams. A person whose email failed has an error trace of their own, and the others still receive theirs.

::screenshot{id=openaev-phishing-execution-details alt="The Execution details tab of the phishing inject Fake EHS 2026 group booking, Executed, with its trace INFO Phishing emails sent to 5 target(s)."}

| Error | Cause |
| --- | --- |
| **Phishing inject requires an email template** | No email template was chosen |
| **Phishing email template not found** | The template was deleted after the inject was created: choose another one |
| **Phishing landing page not found** | The landing page behind the action no longer exists |
| **Phishing needs at least one target user** | The targeted teams hold nobody |

A person who clicks while the others are still being sent is counted: their link works from the moment their email leaves.

## What the person receives

The link in the email reads `https://` followed by the host and `/auth/` and a long random token, such as `https://reservations.grand-meridian.example/auth/q3V9…`. The host is the [custom domain](/openaev/use/phishing/custom-domains/) of the landing page, or the platform's address. Nothing in it names OpenAEV, the tenant or the exercise. Links of emails sent before an upgrade keep working.

The link opens the landing page, which turns **Link not clicked** red. Submitting its form turns **Credentials not submitted** red, then takes the person to the landing page's redirect URL, if it has one.

:::example[Send the fake summit booking to the French and Spanish front desks]
Grand Meridian Hotels runs its front-desk awareness exercise on 12 April 2026, with the lure Velvet Harbor sent to Paul Garnier's night shift. In the scenario **Front-desk phishing**, the awareness manager adds the inject:

| Field | Value |
| --- | --- |
| Action | **Phishing: EHS 2026 booking portal** |
| **Title** | `Fake EHS 2026 group booking` |
| **Scheduling** | **Days** `0`, **Hours** `0`, **Minutes** `0`: the emails leave as soon as the simulation starts |
| **Targeted teams** | `Front desk France`, `Front desk Spain` (5 people) |
| **Email template** | `EHS 2026 group booking request` |
| **Inject expectations** | **Email not opened**, **Link not clicked** and **Credentials not submitted**. **Detection** and **Prevention** are removed: no **Email Security** platform reports to OpenAEV yet |

The manager launches the simulation. A minute later, **Execution details** reads **Phishing emails sent to 5 target(s)**.
:::

## What's next

- [Read who opened, clicked and submitted](/openaev/use/phishing/results/).
- [Inject status](/openaev/use/injects/status/): what the statuses and traces of an inject mean.
- [Findings](/openaev/use/findings/): the credentials the exercise captured.
