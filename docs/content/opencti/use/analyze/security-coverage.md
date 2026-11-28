---
title: Measure your defenses against a threat
description: Measure which techniques and vulnerabilities of a threat your defenses prevent and detect, tested on a schedule by OpenAEV or entered from your own tests, and follow the scores in OpenCTI.
sidebar_position: 6
doc_type: how-to
products:
  - opencti
  - openaev
personas: [soc-manager, cti-analyst, risk-manager]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/security-coverage/
---

A security coverage tells the SOC, for one threat, how much of its attack patterns and vulnerabilities the defenses prevent and detect: which ATT&CK techniques of Velvet Harbor your EDR blocks, which ones your SIEM sees, and which ones nothing catches. OpenAEV can test them on your assets on a schedule and send the scores back to OpenCTI, so the coverage stays current as your defenses change; or you enter the results of tests run elsewhere, such as a red team exercise. This page shows how the two modules stay in sync, how to create a coverage by hand or automatically, and how to read the scores.

## How automated coverage works

::diagram{id=opencti-security-coverage-loop}

OpenCTI sends the coverage and the threat to OpenAEV as a STIX bundle, through the OpenAEV connector. OpenAEV builds a scenario with one inject per technique it can test on your assets, runs it at the recurrence you set, and sends the results of the last simulation back.

:::only{opencti=">=7.261030"}
OpenCTI keeps one set of results per OpenAEV instance, so two OpenAEV tenants testing the same threat do not overwrite each other.
:::

:::only{opencti="<7.261030"}
The coverage holds one set of scores: each new simulation replaces the previous scores, whichever OpenAEV tenant sent them.
:::

Three rules to plan with:

- **One coverage per threat.** A report, a campaign or an intrusion set has at most one security coverage. Creating another one for the same threat updates the existing one and keeps its results.
- **A score reflects what ran.** Each score, from 0 to 100, comes from the injects that returned a result. It can be partial when some injects did not run on your assets, or when OpenAEV had no test for a technique.
- **The threat sets the scope.** OpenAEV tests the attack patterns and vulnerabilities linked to the covered entity when the coverage is sent. When the threat gains new techniques, refresh the coverage, as described in [When a technique shows no score](#when-a-technique-shows-no-score).

:::only{opencti=">=7.261127"}
The tested entities also follow the threat when it loses some. Remove an entity from the covered report, grouping or incident response, or delete a **uses** or **targets** relationship from the covered entity to an attack pattern or a vulnerability. OpenCTI then deletes the **has covered** relationships of the coverage results to that entity, unless the threat still reaches it through another link. The results themselves stay.
:::

## Before you start

- Your role has the **Create / Update knowledge** capability. Deleting a coverage also takes **Delete knowledge**.
- The threat to cover is a report, a grouping, an incident response, an incident, a campaign or an intrusion set, and it is linked to the attack patterns or vulnerabilities to test.
- For automated coverage, an administrator has connected OpenAEV to OpenCTI. The **OpenAEV Coverage** connector then appears among the [connectors](/opencti/use/ingest/connectors/) of the platform. Each OpenAEV tenant you connect brings its own connector. See [Enrich OpenCTI security coverage](/openaev/use/coverage-enrichment/).

## Create a security coverage

:::steps

1. Open the threat, for example a report, and click **Add Security coverage** on its overview. You can also start from :menu[Analyses > Security coverages] with **Create Security Coverage**; the form then adds a **Select entity to cover** step.
2. On the **Choose type** step, choose how to assess the coverage:
   - **Automated using enrichment** lets OpenAEV test it. The option is greyed out until an enrichment connector for security coverage is available.
   - **Manual Input** lets you enter results from tests run elsewhere.

   ::screenshot{id=opencti-security-coverage-mode alt="The Create a security coverage panel offering two choices: Manual Input and Automated using enrichment, with a sentence under each."}

3. Fill in the details. The **Name** comes from the covered entity; change it if you want.
   - For automated coverage, set the **Coverage recurrence (every x)**, daily by default; the **Duration**, 30 days by default; the **Type affinity**, **Endpoint**; and the **Platform(s) affinity** of the assets to test on, Windows, Linux and macOS by default.
   - For manual coverage, follow the steps below.
4. Click **Create**.

:::

:::example[Test the defenses of the hotel front desks every week]
The report "Velvet Harbor targets hotel booking systems" lists the techniques Velvet Harbor uses against front-desk workstations: spearphishing attachments, PowerShell and web protocols for command and control. OpenAEV is connected and the front-desk workstations run Windows. Lionel, the SOC manager, opens the report, clicks **Add Security coverage**, chooses **Automated using enrichment**, and sets:

- **Name**: `Coverage of Velvet Harbor targets hotel booking systems`, as proposed
- **Coverage recurrence (every x)**: one week, so that a change on the workstations shows in the next week's scores
- **Duration**: the default, 30 days
- **Type affinity**: **Endpoint**
- **Platform(s) affinity**: `windows` only, since the front desks run nothing else

He clicks **Create**. OpenAEV builds a scenario tagged `security coverage: windows`, with one inject per technique on the asset groups its asset rule for that tag names, and the first scores appear on the **Overview** tab after the first run. Every week after that, the scores show whether the SOC still detects each technique.
:::

::::only{opencti=">=7.261030"}
### Enter manual results

:::steps

1. On the **Select entities to test** step, pick the attack patterns, vulnerabilities, indicators, artifacts or security platforms you tested. The entities of the covered threat are offered first.
2. On the **Coverage Result details** step, set the **Coverage validity period**, then add a score from 0 to 100 for each metric you measured: prevention, detection or vulnerability. Add a **Source external link** to the test report if you have one.
3. Click **Create**. OpenCTI links each tested entity to the result in a background task, so the results can take a moment to appear.

:::

:::example[Record a red team exercise on Velvet Harbor techniques]
An external red team replayed the Velvet Harbor attack against a hotel group in January 2026: the endpoint protection blocked 6 of 10 phishing attachments, and the SOC detected 8 of 10 runs. On the report "Velvet Harbor targets hotel booking systems", click **Add Security coverage**, choose **Manual Input**, then:

1. Under **Select entities to test**, keep the three attack patterns of the report: **Spearphishing Attachment**, **PowerShell** and **Web Protocols**.
2. Under **Coverage Result details**, set the **Coverage validity period** to three months, the time until the next exercise, then add two scores: `prevention` at `60` and `detection` at `80`. In **Source external link**, enter `https://docs.example.com/red-team/2026-01-velvet-harbor.pdf`.
3. Click **Create**.

The **Overview** tab shows 60 for prevention and 80 for detection, and the matrix colors the three techniques. After the next exercise, add its scores as a new result to compare the two rounds.
:::

### Add or correct a result later

Each round of tests is a result of its own, so the coverage keeps its history.

:::steps

1. Open the coverage, then its **Result** tab, and click **Add manual Coverage Result**: the **Create Security Coverage Result** panel opens. To correct an existing result instead, open the menu next to the button and choose **Update existing Security Coverage Result**: the **Update a security coverage result** panel starts with **Select result**.
2. In **Result details**, give the result a **Name**, its **Valid from** and **Valid to** dates and at least one score, each between 0 and 100. A name or a source link already used by another result of the coverage is refused. **Create** records the result without tested entities; click **Next** to link them.

   ::screenshot{id=opencti-security-coverage-result-form alt="The Create Security Coverage Result panel on its Result details step, with the Name field, a coverage score, the Source external link and the validity dates."}

3. In **Select entities to test**, pick the entities the round tested in the **Entities to test coverage** list, then click **Next**.
4. In **Coverage scores**, OpenCTI shows the link it creates: the result on the left, **has covered**, and the tested entity on the right, or **Multiple entities selected** when you picked several. Give the scores the tested entities reached, then click **Create**. These scores apply to every entity you picked: to score some entities differently, add them afterwards with **Update existing Security Coverage Result**.

:::
::::

::::only{opencti="<7.261030"}
### Enter manual results

On the details step, choose the metrics you assessed, detection, prevention or vulnerability, and give each a score from 0 to 100. To change the scores later, click **Update** at the top right of the coverage.

:::example[Record a red team exercise on Velvet Harbor techniques]
An external red team replayed the Velvet Harbor attack against a hotel group in January 2026: the endpoint protection blocked 6 of 10 phishing attachments, and the SOC detected 8 of 10 runs. On the report "Velvet Harbor targets hotel booking systems", click **Add Security coverage** and choose **Manual Input**. On the details step, set the **Coverage validity period** to three months, the time until the next exercise, and add two scores: the **Coverage name** `prevention` with the **Coverage score (0-100)** `60`, then `detection` with `80`. Click **Create**: the **Overview** tab shows both scores.
:::
::::

## Cover every new threat automatically

With the Enterprise Edition, a [playbook](/opencti/use/automate/playbooks/) can create the coverage for you, so every new threat that matters to you is tested without anyone asking. Its coverage step takes the same settings as the form: **Coverage recurrence (every x)**, **Duration**, **Type affinity** and **Platform(s) affinity**, plus **Force manual coverage (prevent enrichment connectors from running)** to create the coverage without sending it to OpenAEV. A **Send for ingestion** step must follow it.

:::example[Test every new Velvet Harbor report]
The SOC wants a coverage for each new report about Velvet Harbor, tested every week on the Windows front desks. A team lead with **Manage Playbooks** builds a playbook with three components:

| Component | Settings |
| --- | --- |
| **Listen knowledge events** | **Create** on. **Filters**: **Entity type** = Report and **Label** = `velvet-harbor` |
| **Create Security Coverage** (**Security coverage** before 7.260811) | Scope on the main element, the report. **Coverage recurrence (every x)**: one week. **Duration**: 30 days. **Type affinity**: **Endpoint**. **Platform(s) affinity**: `windows`. **Force manual coverage (prevent enrichment connectors from running)** off |
| **Send for ingestion** | No settings |

Each new report labelled `velvet-harbor` gets its coverage within seconds, and OpenAEV sends the first scores after its first run.
:::

## Read the results

The **Overview** tab summarizes the coverage:

- **Coverage information** gives the date of the **Last result**, its **Valid from** and **Valid until** dates, and one gauge per metric: the share of **Success** and **Failure**. A grey gauge marked **Unknown** has no score yet; **Empty coverage** means no metric was measured.
- The covered entity, the number of tested entities and the score of each vulnerability.
- **Attack patterns coverage** colors each technique of the threat by its score, on the MITRE ATT&CK matrix (**Matrix view**) or as a kill chain list (**Lines view**). **Display only used techniques** hides the techniques the threat does not use; **Display the whole matrix** shows them again.

::screenshot{id=opencti-security-coverage-overview alt="The overview of a security coverage with the prevention and detection scores, the count of tested entities, and the attack patterns colored by coverage on the matrix."}

:::only{opencti=">=7.260811"}
The **Tested entities** chart compares the entities that were tested with those **Not covered** by any test.
:::

::::only{opencti=">=7.261030"}
The **Result** tab lists each tested entity with its type, name, labels and markings, its coverage score and the result it comes from. Pick a result in **Security coverage results** when there are several. A score shows how much an entity was involved in the OpenAEV scenario, so it can be partial when some injects did not run, when placeholders were not resolved, or when the platform of the assets does not support an action. Hover the **Info** icon above the list for this reminder.
::::

:::only{opencti=">=7.261127"}
The filters of the **Result** tab narrow the list to what you look for: **Coverage**, to keep the entities under a score, such as a `Detection` score below 50; **Source entity**, to keep one result; **Target entity** and **Target type**, to keep some tested entities; and **Modification date**. **Source entity** and **Target entity** offer only the results and tested entities of this coverage.
:::

::::only{opencti="<7.261030"}
:::only{opencti=">=7.260811"}
The **Result** tab lists each tested entity with its coverage score. A dash means no result is available yet.
:::
::::

:::only{opencti=">=7.261127"}
To see the details in OpenAEV, click **Go to OpenAEV** in the header: the link of the results opens in a new tab, the OpenAEV scenario for an automated coverage. The button shows once a result carries a link, and the **Source external link** of a manual result counts as one. When the results carry several links, for example from two OpenAEV instances, the button opens **Select the OpenAEV instance** instead: pick a link, named after its result or the host of its address.
:::

:::only{opencti="<7.261030"}
To see the details in OpenAEV, click **Go to OpenAEV** in the header. The button reads **Provisioning OpenAEV** and stays disabled until OpenAEV has received the coverage.
:::

## When a technique shows no score

A dash instead of a score means OpenAEV created a placeholder inject rather than a test. This happens when OpenAEV does not know the attack pattern, for example a custom technique, or has no payload for it on the platforms of your assets. In OpenAEV:

1. Check that the asset rules map the `security coverage: <platform>` tag of each platform you test, such as `security coverage: windows`, to the right asset groups. See [Set default asset groups by scenario tag](/openaev/configure/asset-rules/).
2. Search the payloads by the MITRE ATT&CK ID of the technique. If none fits your platforms, enable more payload [collectors](/openaev/use/collectors/), such as the curated payloads of Filigran or Atomic Red Team, or create a payload in the [threat arsenal](/openaev/use/threat-arsenal/).

Then, back in OpenCTI, open the coverage, click **Enrichment** in its header and click **Refresh the knowledge using this connector** on the OpenAEV connector. OpenAEV rebuilds the scenario from the current threat, new techniques included. The same panel lists every past request with its status, which helps when a run failed. See [Generate a scenario from OpenCTI security coverage](/openaev/use/scenarios/from-opencti/) for the details.

## What's next

- [Threat-informed validation](/workflows/threat-informed-validation/), the full loop from a threat in OpenCTI to tested defenses in OpenAEV.
- [Generate a scenario from OpenCTI security coverage](/openaev/use/scenarios/from-opencti/), on the OpenAEV side.
- [Follow your priority threats with PIRs](/opencti/use/analyze/pir/), to choose which threats to cover first.
