---
title: Read the result of an inject
description: Open the result of an atomic test or of a simulation inject, read which targets prevented or detected it, follow its execution, and see the findings and detection rules it produced.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, soc-analyst]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/inject-result/
---

The result page of an inject answers one question per target: did your defenses prevent the action, did they detect it, and did it run at all? It also shows what the action discovered and the detection rules that would catch it. This page finds the inject, reads its scores and each target's results, and follows its execution. If you are new to injects and expectations, read [Injects and expectations](/openaev/concepts/injects-expectations/) first.

::diagram{id=openaev-injects-results-layout}

## Before you start

- Your role has **Access assessment**, or access to the simulation. The **Remediations** tab requires the Enterprise Edition.
- The inject has run: an [atomic test](/openaev/use/atomic-testing/) you launched, or an inject of a [simulation](/openaev/use/simulations/) that has started.

## Find the inject

:::steps

1. Open the list that holds it:
   - an atomic test: :menu[Atomic testings];
   - an inject of a simulation: :menu[Simulations], the simulation, its **Overview** tab, then the **Injects results** list at the bottom.
2. Narrow the list. Type part of a title in the search bar, or add a filter: **Inject title**, **Type**, **Attack pattern**, **Kill chain phase**, **Domains**, **Tags**, **Assets**, **Asset groups**, **Teams** or **Updated at**. The list keeps your filters the next time you open it.
3. Read the line: **Execution status**, **Target**, **Global score** and **Start time** tell you which inject to open. A disabled inject shows **Disabled** and never runs.
4. Click the line. The result page opens on its **Overview** tab.

:::

The page of a team and of an organization lists the injects that targeted them, with the same columns and filters. These lists sort on **Name** and **Updated** only, and none filters on the execution status: scan the **Execution status** column.

## Read the scores

The header shows the status of the inject, its start date, end date and duration, the platforms of its action and its kill chain phases. When the status is **Error** or **Partial**, hover it to read the first error message. The meaning of every status is in [Inject execution statuses](/openaev/use/injects/status/).

Under it, one tile per expectation type sums up every target: **Prevention**, **Detection**, **Vulnerability** and **Human response**. Each tile gives the overall result, such as **Prevented**, **Partially Detected** or **Pending**, and a bar of how many targets reached each result. A type the inject does not expect reads **No expectation for** that type. How the platform computes these results is in [How expectations score an inject](/openaev/use/expectations/).

## Read the results of each target

:::steps

1. In **Targets**, choose a tab: **Asset groups**, **Teams**, **Assets**, **Persons**, **Agents** or **Manual targets**, the addresses typed by hand for a network scan such as Nmap or Nuclei. Only the kinds of target the inject reached show. An inject without any target, such as an HTTP request, shows no tab: **Targets** reads "This inject has no target. Its results are in the Execution details tab." The page remembers the last tab you opened for this inject.
2. Click a target. **Results by target** shows its results; the arrows at its top move to the previous or next target.
3. Read a section per expectation type first. Each card gives the name of the expectation, its **Score**, and, on a group, its **Validation rule**. For prevention and detection, each security platform that answers has a line, with its **Status**, its **Detection time** and its **Alerts**. Click the number of alerts to see the alerts the collector matched; an alert that carries a link opens in its security platform, in a new tab. A platform's line fills in only after its collector's next run, so allow a few minutes after the inject before you read a missing alert as a miss.
4. Read **Execution**: the traces of the target, one line per step, with the status of each agent. On an asset group, open one of its assets to see them.
5. Read **Terminal view**, on an endpoint or an agent: what each step printed.

:::

::screenshot{id=openaev-injects-results-target alt="The Targets of the atomic test Booking database export on booking-db-01, its one asset selected, and its Results by target: the attack started, then failed, Meridian EDR prevented and detected it, each with 1 alert and a score of 100."}

A card that reads **No automated security platform detected** has no collector able to answer it: OpenAEV failed it as soon as the inject ran. Connect a collector for that kind of security platform, or record the result yourself, as [Add expectation results by hand](/openaev/use/expectations/validation/) explains.

## Follow the execution and the action

| Tab | What it shows | When it shows |
| --- | --- | --- |
| **Overview** | The targets and their results | Always |
| **Execution details** | The traces of the inject as a whole: its distribution to the agents, a late start, a timeout | Always |
| **Action details** | The action that ran: its **Payload**, its **Commands** with their arguments and prerequisites, and its **Output parser** | When the action comes from the Threat Arsenal |
| **Findings** | What the output parser extracted from the output: open ports, accounts, vulnerabilities. Filter them by type, date, asset or asset group. See [Review what your injects discovered](/openaev/use/findings/) | When the action can produce findings |
| **Remediations** | One tab per security platform, each with the **Detection Rule** that would catch this action | When the action comes from the Threat Arsenal. Enterprise Edition |

::screenshot{id=openaev-injects-results-execution-details alt="The Execution details tab of the atomic test: its start and end dates, its duration of 10 minutes 30 seconds, and the trace that distributed the inject to 1 agent on 1 endpoint."}

::screenshot{id=openaev-injects-results-remediations alt="The Remediations tab with a tab per security platform, Meridian EDR and Meridian email gateway: the Detection Rule of Meridian EDR, a CrowdStrike Falcon event search on the booking database servers, marked Rules written by Human, with the Generate with AI button greyed out."}

A detection rule is marked **Rules written by Human**, **Rules generated by AI**, or **Payload changed since rule was edited** when the action changed after the rule was written: review it before you deploy it. A platform with no rule yet reads **No detection rule available for this security platform yet.**: write the rule on the action, in the Threat Arsenal, or generate it as the card below explains. Without any security platform, the tab asks you to create one first, manual platforms included.

:::xtm-one[Generate a detection rule with AI]
On the **Remediations** tab, **Generate with AI** asks an XTM One agent to write the detection rule of a security platform: the agent chosen in the list next to the button. It works for CrowdStrike and Splunk platforms, on actions of the **Command** and **DNS resolution** types, while the rule is still empty. It requires the Enterprise Edition. Without XTM One, the same button reads **Use Ariane** and uses the AI configured in OpenAEV. How to connect XTM One is in [Connect the modules](/deploy/configure/connect-modules/).
:::

:::example[Check what Meridian EDR saw of the booking database export]
On 12 April 2026, the validation engineer of Grand Meridian Hotels replays the export Velvet Harbor ran on the booking database server, as the atomic test **Booking database export on booking-db-01**. Once it has run, they open :menu[Atomic testings] and click its line, which reads **Error**.

On the result page, the tiles read **Prevented** and **Detected**. Under **Targets**, the **Assets** tab lists `booking-db-01`: **Meridian EDR** prevented and detected the command, with 1 alert each, at the minute the attack started. Its **Execution** reads **Timeout**: Meridian EDR killed the implant, which never reported back, so the inject ends in **Error** although the defense held. The engineer opens **Remediations** and checks the **Detection Rule** the detection team wrote for Meridian EDR, so that it also catches the export on the other booking servers.
:::

## What's next

- [Inject execution statuses](/openaev/use/injects/status/): every status of the header and of the traces.
- [Add expectation results by hand](/openaev/use/expectations/validation/): when no collector reports.
- [Review what your injects discovered](/openaev/use/findings/).
- [Simulation reports](/openaev/use/simulations/reports/): share the results.
