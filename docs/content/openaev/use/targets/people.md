---
title: Organize the people your injects target
description: Create the persons who receive emails, SMS and challenges, group them in teams, sort them by organization, and choose which persons of a team a scenario reaches.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/people/
---

The human side of a simulation needs people to reach: the front-desk employees who receive a phishing email, the crisis cell that gets a journalist's call, the SOC that must answer a manual expectation. In OpenAEV, each of them is a person, persons are grouped in teams, and an inject always reaches persons through their teams. Organizations sort persons and teams by company or subsidiary. This page creates them and adds them to a scenario. Why teams and not persons is explained in [How injects reach assets and people](/openaev/use/targets/).

::diagram{id=openaev-people-model}

## Before you start

- Your role has **Access teams & persons** to see :menu[Persons] and :menu[Teams], **Manage teams & persons** to create and change them, and **Delete teams & persons** to delete them.
- :menu[Organizations] needs **Access tenant settings** to be seen and **Manage tenant settings** to be changed.

## Add the persons

:::steps
1. Open :menu[Persons] and click **Create**. The **Create a new person** panel opens.
2. Enter the **Email address**. Emails and phishing injects go there, and it identifies the person: you cannot change it later.
3. Enter the **Firstname** and **Lastname**, then choose the **Organization** and the **Country**.
4. Enter the **Phone number (mobile)** if the person must receive SMS injects, starting with a plus sign and the country code. A **Phone number (landline)** is optional.
5. Paste the **PGP public key** if the person must receive encrypted emails.
6. Add **Tags**, then click **Create**.
:::

To keep the persons in step with your directory instead of typing them, run a collector that imports them, such as Microsoft Entra or Google Workspace: see [How collectors measure what your security tools saw](/openaev/use/collectors/).

A person is a user of OpenAEV without a password: they receive injects, and sign in only if an administrator gives them access. When the email address already belongs to a user, **Create** adds that user to your persons and keeps their own details. To change or delete a person, open the menu at the end of their line; the platform administrator account can be updated, never deleted. **Export this list** saves the persons as a CSV file.

::screenshot{id=openaev-person-create alt="The Create a new person panel: email address, first and last name, organization, country, phone numbers, PGP public key and tags."}

## Group them in teams

:::steps
1. Open :menu[Teams] and click **Create**. The **Create a new team** panel opens.
2. Enter the **Name**: two teams of the list cannot share one. Add a **Description**, the **Organization** and **Tags**.
3. Click **Create**.
4. Open the menu at the end of the team's line and choose **Manage persons**.
5. Click **Add players**, select the persons, and click **Add**.
:::

In **Manage persons**, the **Options** column shows three icons for each member: an email address, a PGP key and a phone number, green when the person has one and orange when not. Check it before a simulation: a person without a phone number cannot receive SMS injects, and one without a PGP key cannot receive encrypted emails. The members are sorted by **Email address**; click the **Organization** header to sort them by the name of their organization instead, for example to review one subsidiary's front desk at a time.

The list shows each team with the number of its **Persons**. Open a team to see its members, the scenarios and simulations it takes part in, and its results.

A team created from inside a scenario or a simulation, with **Only in this context** checked, exists there only: it does not appear in :menu[Teams], and its name only has to be unique in that scenario. Use it for a role that makes sense in one exercise, such as the fake journalists of a crisis scenario.

::screenshot{id=openaev-teams-list alt="The Teams list: Crisis cell, Front desk France and Front desk Spain of Grand Meridian Hotels, with 3, 3 and 2 persons and the front-desk tag on the two front desks, among other teams such as CEO and Head of Communications."}

## Sort them by organization

An organization groups the persons and teams of one company or subsidiary, such as each hotel chain a CERT serves.

:::steps
1. Open :menu[Organizations] and click **Create**. The **Create an organization** panel opens.
2. Enter the **Name**, a **Description** and **Tags**, then click **Create**.
3. Set the organization in the **Organization** field of its persons and teams.
:::

The organization's page lists its **Members**, its **Teams**, and the findings and injects that concern them. An organization belongs to one tenant, and a person who belongs to several tenants has one organization in each: setting it in one tenant leaves their organization in the others unchanged. An organization sorts and filters; it does not hide anything. To keep the people, scenarios and results of two subsidiaries apart, give each its own tenant: see [Give each organization its own tenant](/openaev/administer/multi-tenancy/).

## Add a team to a scenario

An inject can only target the teams of its scenario or simulation.

:::steps
1. Open the scenario and click **Configuration** in its header. The **Scenario configuration** panel opens on its **Teams** tab.
2. Add the teams the scenario reaches.
3. Click a team to see its persons. All of them are **Enabled** when the team joins the scenario: click **Enabled** beside a person to switch it to **Disabled**, and that person no longer receives the scenario's injects.
:::

The injects of the scenario can now target these teams under **Targeted teams**, or every team of the scenario with **All teams**. Each enabled person gets their own expectations, and so does the team. When the scenario is launched, its simulation copies the teams and their enabled persons as they are at that moment.

:::example[Prepare the front desk for a phishing exercise]
Grand Meridian Hotels wants to know whether its front desks report a fake booking request like the one Velvet Harbor sent on 9 January 2026. The exercise planner creates:

| What | Values |
| --- | --- |
| Organization | **Name** `Grand Meridian Hotels` |
| Person | **Email address** `paul.garnier@grand-meridian.example`, **Firstname** `Paul`, **Lastname** `Garnier`, **Organization** Grand Meridian Hotels, **Country** France, **Phone number (mobile)** `+33600000001`, **Tags** `night-shift` |
| Team | **Name** `Front desk France`, **Organization** Grand Meridian Hotels, **Tags** `front-desk`, with Paul Garnier, Camille Roux and Julien Bernard |
| Team | **Name** `Front desk Spain`, **Organization** Grand Meridian Hotels, **Tags** `front-desk`, with Lucía Ortega and Javier Morales |

In the scenario `Velvet Harbor replay`, the planner adds both teams, and disables in **Front desk France** the employee on leave that week. The email inject **Fake summit booking request** targets both teams, with the manual expectation `Reported to the SOC`: each front-desk employee must report the email to the SOC within the hour.
:::

## What's next

- [How a phishing exercise works](/openaev/use/phishing/): send a lure to your teams and score who reports it.
- [Add the assets your simulations target](/openaev/use/targets/assets/): the technical side of the targets.
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/): how the reactions of persons are scored.
