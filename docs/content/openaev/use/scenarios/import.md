---
title: Import a timeline of injects from a spreadsheet
description: Describe your spreadsheet once in an XLS mapper, test it, then import a crisis exercise timeline into a scenario as emails and SMS with their delays and teams.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/scenario-import/
---

Crisis exercises are often written in a spreadsheet: one row per message, with its time, the team that receives it and its text. OpenAEV imports such a timeline into a scenario, as email and SMS injects, so you do not type it again. An XLS mapper tells OpenAEV how to read your spreadsheet: which column gives the kind of inject, and which columns fill its fields. You create the mapper once, test it, and reuse it for every exercise written on the same model. For what a scenario holds, see [Build a scenario that replays an attack](/openaev/use/scenarios/build/).

::diagram{id=openaev-scenarios-import-mapper}

## Before you start

- To create and test a mapper, your role has **Manage tenant settings**.
- To import, you only need the right to manage the scenario or simulation: **Manage assessment**, or a grant on it. Every member of the tenant can pick an existing mapper, so the person who imports does not need the settings.
- The spreadsheet is an `.xlsx` or `.xls` file of less than 25 MB, with one row per inject and one column that says which kind of inject the row is.
- Emails and SMS can be sent: the import creates **Send individual mails**, **Send multi-recipients mail** and **Send a SMS** injects, the only ones it supports. See [Injects](/openaev/use/injects/).

## Describe the spreadsheet in a mapper

:::steps

1. Open :menu[Settings > Data ingestion > XLS mappers] and click **Create**. The **Create a xls mapper** panel opens.
2. Give a **Mapper name**, and choose the **Inject type column**: the letter of the column that says which kind of inject a row is.
3. Next to **Representation for inject type**, click **Add**. A representation describes one kind of inject.
4. In **Matching type in the xls**, enter the text that marks this kind in the inject type column, such as `mail`. It is a regular expression, case sensitive, that matches anywhere in the cell: `mail` matches `Send mail`, not `Mail`.
5. In **Inject type**, choose the inject to create, such as **Send individual mails**. The rules of its fields appear: **Title**, **Description**, **Trigger time**, then the fields of the inject, such as **Teams**, **Email subject** and **Email body**, then **Expectation name**, **Expectation description** and **Expectation score**. A red star marks the mandatory ones.
6. For each field, enter the column letter in **Rule attributes columns**. To join several columns, separate the letters with `+`. To give a value to every row, click the gear and fill in **Default value**: it applies when the cell is empty.
7. Add a representation for each other kind of inject, such as `(sms|SMS)` for **Send a SMS**.
8. Click **Create**.

:::

::screenshot{id=openaev-scenarios-import-mapper alt="The Update the xls mapper panel of Grand Meridian exercise timeline: inject type column A, and its representation Send individual mails, matching mail, with the Title in column B, no Description and the Trigger Time in column C."}

A row whose inject type matches no representation is skipped. A row that matches two representations is skipped too: make the expressions exclusive.

### When the injects happen

The **Trigger time** column gives the time of each inject, in one of two ways:

- **Relative**, from the first inject: `D` or `J` for days, `H` or `T` for hours, `M` for minutes, each followed by a signed number. `D+1 H+3 M+45` is one day, three hours and forty-five minutes after the start. A letter without a number counts as zero. A day can also take a time of day, `D+1 09:30`, which then needs a start date.
- **Absolute**: a date and time, such as `2026-03-12T09:30:00`, or a time of day, such as `09:30`. For another format, click the gear of **Trigger time** and set a **Time pattern**, in the notation of Java date formats, such as `dd.MM.yyyy HH'h'mm`.

OpenAEV turns the times into delays from the earliest one. A timeline with times of day, such as `09:30`, needs a start date: the import asks for it, and sets the scenario to run once on that day, at the earliest time. A timeline that mixes the two places each relative row on the nearest absolute row.

### Who receives each inject

The **Teams** column holds team names, separated by commas. A name that matches an existing team links that team. Any other name creates a team that exists only in this scenario, and the test reports it with **Team not found (contextual team will be created)**. To target all the teams of the scenario, click the gear of **Teams** and set the **All teams value**, the text that means everyone, such as `All`.

### Test the mapper

:::steps

1. In the mapper's panel, click **Test**. The **Testing XLS mapper** dialog opens.
2. Choose the spreadsheet and click **Next**.
3. Choose the **Sheet** and the **Timezone** in which to read the times, then click **Test**.

:::

**Result** shows the injects the mapper would create; **Log** shows the rows it skipped and why. Nothing is saved. Fix the rules until every row you expect is in **Result**.

## Import the timeline into a scenario

:::steps

1. Open the scenario, then its **Injects** tab. A simulation's **Injects** tab imports the same way.
2. Click **Import injects**, the upload icon, and choose **XLS import (mapper)**.
3. Choose the spreadsheet, then click **Next**.
4. Choose the **Sheet**, the **Mapper** and the **Timezone**. When the times are times of day only, a **Start date** field appears: set the day of the exercise.
5. Check the summary, which counts the injects ready to import and lists the errors by column and by row, then click **Launch import**.

:::

::screenshot{id=openaev-scenarios-import-dialog alt="The Import injects dialog on its first step: Select your file, an XLS spreadsheet, before Next."}

The injects appear on the **Injects** tab, with their delays, teams and expectations. The spreadsheet stays on the server for 60 minutes, so you can change the mapper and import again in that time without choosing the file again.

:::example[Import the board's crisis exercise on the guest data breach]
Hélène Marchetti, CISO of Grand Meridian Hotels, asks for a tabletop exercise for the board: the guest records stolen by Velvet Harbor are offered for sale, and the press calls. The exercise planner wrote the timeline in `guest-breach-exercise.xlsx`:

| A, Channel | B, Action | C, Time | D, Teams | E, Subject | F, Content |
| --- | --- | --- | --- | --- | --- |
| `mail` | Alert from the SOC | `D+0` | Executive committee | Guest records offered for sale | The SOC found 2.4 million guest records of Grand Meridian on a criminal forum. |
| `mail` | Journalist asks for comment | `D+0 H+1` | Head of Communications | Press request | A journalist of a travel magazine asks whether guest data leaked. |
| `note` | Facilitator checks the decisions | `D+0 H+2` | | | |

The mapper `Grand Meridian exercise timeline` has **Inject type column** `A` and one representation:

| **Matching type in the xls** | **Inject type** | Rules |
| --- | --- | --- |
| `mail` | **Send individual mails** | **Title** `B`, **Trigger time** `C`, **Teams** `D`, **Email subject** `E`, **Email body** `F` |

The test shows two injects; the `note` row is skipped, and **Executive committee** and **Head of Communications** are reported as teams to create. On the scenario `Guest data breach - board exercise`, the import creates the two injects at 0 and 60 minutes, and the two teams for this scenario only.
:::

## What's next

- [Build a scenario that replays an attack](/openaev/use/scenarios/build/), to set the teams and launch the exercise.
- [Run a simulation and read its results](/openaev/use/simulations/), to follow the players' answers.
- [People](/openaev/use/targets/people/), to add the persons of the teams created by the import.
