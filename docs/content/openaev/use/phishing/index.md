---
title: How a phishing exercise works
description: What an OpenAEV phishing exercise measures, the landing pages and lure emails it is built from, how each person's open, click and submission is scored, and what it needs.
doc_type: explanation
products:
  - openaev
personas: [exercise-planner, validation-engineer, admin]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/build/components/phishing/overview/
---

A phishing exercise sends your own people a fake email and measures, person by person, who opened it, who followed its link and who typed their credentials into the page it leads to. OpenAEV runs it on its own: it writes the email, hosts the page, sends through the platform's mail server and scores every step, with nothing else to install. This page explains what an exercise is made of, how a person's actions become results, and what the platform needs before the first one. The pages under it walk through each task.

:::warning
Run phishing exercises only against your own users, with the authorization your organization requires. You are responsible for obtaining it before any exercise.
:::

## What an exercise is made of

::diagram{id=openaev-phishing-exercise}

A phishing exercise is an inject like any other, in a scenario, a simulation or an atomic test. Two reusable components make it a phishing inject:

| Component | What it is | Where you build it |
| --- | --- | --- |
| **Landing page** | The page a person sees after clicking the link: usually a sign-in form. Each landing page becomes an action of the Threat Arsenal, named **Phishing:** followed by its name | :menu[Components > Phishing], tab **Pages**. See [Build the landing page of a phishing exercise](/openaev/use/phishing/landing-pages/) |
| **Email template** | The lure email: its subject, its sender, its body with the link. Any email template can go with any landing page | :menu[Components > Phishing], tab **Emails**. See [Write the lure email of a phishing exercise](/openaev/use/phishing/email-templates/) |

You then add the landing page's action as an inject, choose the email template and the teams that receive it. Every person of those teams gets their own email, with their own link. See [Send a phishing exercise to your teams](/openaev/use/phishing/run/).

The action carries the **E-mail Infiltration** domain and the MITRE ATT&CK techniques Phishing: Spearphishing Link (T1566.002) and Phishing for Information: Spearphishing Link (T1598.003), when the MITRE data is imported in the platform. It targets teams, never assets: no agent or executor takes part.

::screenshot{id=openaev-phishing-pages-list alt="The Phishing component listing the landing pages, the EHS 2026 booking portal next to the Default login page the platform ships."}

## How a person's actions become results

Each inject measures three steps for every person, in the order an attack unfolds:

| Step | Turns red when the person |
| --- | --- |
| **Email not opened** | Opens the email and loads its images (a tracking pixel) |
| **Link not clicked** | Opens the landing page from the link |
| **Credentials not submitted** | Submits the form of the landing page |

The names state the outcome you want, so a step is green while it stays true and red once the person falls for it. Every step is green when the email leaves, and nothing expires: a person who never reacts keeps three green steps. A later step also turns the earlier ones red, since a person who submits the form has opened the email.

Mail security gateways open every link and image of an incoming email to inspect it. OpenAEV ignores a hit that arrives within 10 seconds of the email or that comes from a known scanner, so the inspection does not count as a person's click. What the person submits can be kept as a **Credentials** finding.

Two more expectations come with every phishing inject: **Detection** and **Prevention**, which a security platform of the **Email Security** type fills when it detects or blocks the email. Reading all of it is in [Read who opened, clicked and submitted](/openaev/use/phishing/results/).

## Before the first exercise

- **Sending.** OpenAEV sends through the mail server of its configuration, the same as the email injector: the `spring.mail` keys of the [configuration reference](/reference/openaev/configuration/#spring-mail). Its default sender is `openaev.default-mailer`, in the [`openaev` block](/reference/openaev/configuration/#openaev).
- **Reaching the page.** The link in each email points at the platform's address, `openaev.base-url`, so your people must be able to open it from where they read their email. To use a hostname of your own instead, see [Use your own domain for phishing links](/openaev/use/phishing/custom-domains/).
- **Content to start from.** Each tenant starts with a landing page, **Default login page**, and an email template, **Default lure email**. You can run an exercise with them right away, edit them or delete them.

## Who can do what

Phishing components have their own capabilities, in the role editor:

| Capability | What it allows |
| --- | --- |
| **Access phishing** | See the landing pages and email templates |
| **Manage phishing** | Create, update and duplicate them |
| **Delete phishing** | Delete them |

Adding the inject to a scenario, simulation or atomic test takes **Manage assessment**, and launching it **Launch assessment**. Custom domains are a tenant setting: they take **Manage tenant settings**. Every capability is in the [capabilities reference](/reference/openaev/capabilities/).

## Edition

Phishing exercises are part of every edition. Only **Generate with AI**, which writes the landing page or the email for you, requires the Enterprise Edition and a platform connected to XTM One.

:::xtm-one[Write the lure and the page with XTM One]
When OpenAEV is connected to XTM One, the landing page and email template editors show **Generate with AI**: an AI agent writes the page's HTML and CSS, or the email's subject and bodies, from a preset or your instructions, and refines what is already there. Connecting OpenAEV to XTM One is in [Connect the modules](/deploy/configure/connect-modules/).
:::

## What's next

- [Build the landing page of a phishing exercise](/openaev/use/phishing/landing-pages/)
- [Write the lure email of a phishing exercise](/openaev/use/phishing/email-templates/)
- [Send a phishing exercise to your teams](/openaev/use/phishing/run/)
- [Read who opened, clicked and submitted](/openaev/use/phishing/results/)
- [Use your own domain for phishing links](/openaev/use/phishing/custom-domains/)
