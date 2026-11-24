---
title: Set how strong your controls are in the Defenses step
description: Assess the strength and coverage of your controls by hand, or bring them from Drata, in the Defenses step of Get started, so OpenCRQ knows how much of an attack they stop.
sidebar_position: 5
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Defenses** is the fourth step of :menu[Get started]. Your controls decide how much of an attack is stopped, and a control left unassessed counts as absent: your losses are then overestimated. The step assesses the 51 controls of OpenCRQ's catalogue from two sources, which add up: Drata, and the assessment you give yourself.

## Before you start

- You are a member of the tenant. Assessing a control needs no capability.
- To set up Drata, you have **Manage Security**, and a platform administrator has given OpenCRQ the Drata API key of the tenant. See [Import compliance readiness from Drata](/opencrq/use/sources/drata/#before-you-start).
- You know what each control means for OpenCRQ: its **strength** and its **coverage**. See [What you declare](/opencrq/use/defenses/controls/#what-you-declare).

## The defense sources

Under **Defense sources**, the step reads "Use as many as you need, they add up." Beside it, how many controls of the catalogue the tenant has assessed, such as **12 of 51 capabilities assessed**, and **See all controls**, which opens the list of the controls.

| Tile | What it brings | What it shows |
| --- | --- | --- |
| **Drata** | The strength and the coverage OpenCRQ derives from the Drata controls mapped to each of its controls | The state of the connector: **Unavailable**, **Not configured**, **Never synced**, **Syncing**, **Healthy** or **Failing**, then the progress of a run while one goes. **Configure** opens it |
| **Assess yourself** | The strength and the coverage you declare for each control | How many controls you assessed by hand, such as **3 capabilities assessed by hand**, and **Active** once there is one. **Assess** opens the **Assess yourself** panel |

The **Drata** tile shows no count: OpenCRQ does not tell which assessments Drata fed. A control with one axis set by hand counts on the **Assess yourself** tile, whatever set the other.

The step is started once one control is assessed, and done once the 51 are. Any source counts: your own assessment, Drata, a published framework assessment, or a declaration the **Enterprise** risk scope makes on its own. A control assessed **Not deployed** counts too. A started step is enough for Get started to finish.

## Assess your controls yourself

The panel writes the tenant's assessment of each control: it applies to every risk scope, except where a scope declares its own. It is the assessment the **Controls** page shows. The panel has no **Comment**, and a choice made in it clears the comment of that control. To declare a scope's own posture, see [Declare a control for a risk scope](/opencrq/use/defenses/controls/#declare-a-control-for-a-risk-scope).

:::steps
1. On the **Assess yourself** tile, select **Assess**. The panel opens on the controls grouped by domain, from **Endpoints and servers** to **AI**, with the first domain open. Each domain reads how many of its controls are assessed, such as **0 of 6 assessed**, and each control not yet assessed reads **Not assessed**.
2. On the row of a control, answer **How is the control implemented?**: **Not deployed**, **Basic**, **Intermediate** or **Master**. The icon beside the question, **What each strength rung demands**, gives the criteria of each level for that control.
3. Answer **How widely is it deployed?**: **None**, **Partial**, **Broad** or **Full**. The icon beside it, **What coverage measures**, reminds you that coverage says how much of the concerned perimeter the control is deployed on, not how well it works there.

   ::screenshot{id=opencrq-get-started-defenses-assess alt="The Assess yourself panel of Get started for Grand Meridian Hotels: Endpoints and servers open at 0 of 6 assessed, Endpoint Detection and Response with Intermediate chosen and the note Answer both questions to assess this control, the other controls Not assessed."}

4. Go on with the other controls and domains, then close the panel.
:::

The first assessment of a control needs both answers. After the first one, the row reads "Answer both questions to assess this control." and nothing is saved until you give the other. Once the control is assessed, each choice saves at once, on its own. No message confirms it: the counts of the domain, of the tile and of the step move on.

**Auto** beside a question marks an axis that Drata or a published framework assessment derives. Choosing a level declares that axis by hand instead, and no later sync changes it.

:::example[Assess Grand Meridian's endpoint controls]
Grand Meridian Hotels has not connected Drata yet. Claire Morel, its cyber risk manager, selects **Assess** and starts with **Endpoints and servers**:

- **EDR**, **Endpoint Detection and Response**: **Intermediate**, then **Broad**. The EDR runs on the workstations of the head office and of the hotels, not on every server. After her first choice, the row asks her to answer both questions; the second one saves it.
- **PATCH**, **Patch Management**: **Basic** and **Broad**.
- **MSEG**, **Microsegmentation**: **Not deployed** and **None**. The networks of the hotels are flat, and the control counts as assessed all the same.

The domain now reads **3 of 6 assessed**, the tile **3 capabilities assessed by hand** and **Active**, and the stepper shows **Defenses** started. Grand Meridian France and Grand Meridian Spain keep the postures they declared on their own.
:::

## Bring your controls from Drata

Select **Configure** on the **Drata** tile. The **Drata** panel opens:

- When Drata is not set up for the tenant, the panel reads "A platform administrator enables Drata for this tenant: set the variables below in the platform's environment, then restart the platform." Below it, the variable to set, `CONNECTORS__DRATA__<tenant-id>__API_KEY=`, already holding the id of the tenant. **Copy** copies it, and **See the documentation** opens the platform's configuration on GitHub. The optional variables are in [Before you start](/opencrq/use/sources/drata/#before-you-start) of the Drata page.
- When it is set up and you have **Manage Security**, choose the account the connector runs as under **Service account**, and the **Schedule**, then select **Save**. **Create a new service account**, the default, creates the account with the integration, in the Connectors group: its **Service account name** is filled in with "Drata connector", which you can change. **Use an existing service account** runs the connector as an account the tenant already has. A tenant with no account to reuse is offered the new one only. Below the form, **Sync now** starts a run at once, once a service account is bound and no run is going; "Sync job enqueued successfully." confirms it. **Follow in Integrations** opens the connector in :menu[Integrations].
- Without **Manage Security**, the panel reads "An administrator configures this connector."

Each run sets the strength and the coverage of the controls that Drata controls are mapped to, on the axes left automatic. See [Where it shows, and what it changes](/opencrq/use/sources/drata/#where-it-shows-and-what-it-changes).

## Check that it works

- The count under **Defense sources** gives the controls the tenant assessed, and the **Assess yourself** tile those you assessed by hand.
- **See all controls** lists the controls you assessed with their strength, coverage and effectiveness.
- The stepper shows **Defenses** started, then a check once each of the 51 controls is assessed for the tenant or declared by the **Enterprise** scope.

## What's next

- Select **Next** to go on to the [Finances step](/opencrq/use/get-started/finances/).
- [Import the threats that target your company](/opencrq/use/get-started/threats/), the step before.
- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/), to declare a scope's own posture and read what the tests of OpenAEV show.
- [Import compliance readiness from Drata](/opencrq/use/sources/drata/), to keep the controls in step with your compliance program.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), to see how the controls weigh in the price.
