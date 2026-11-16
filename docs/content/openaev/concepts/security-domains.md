---
title: How security domains group your results
description: The ten security domains of OpenAEV, how an action or an injector gives an inject its domains, and how the Security Domains widget scores each kind of control.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/security-control-validation/
---

A security domain says which kind of control an inject tests: your endpoint protection, your network filtering, your email gateway, the reaction of your teams. Every action of the Threat Arsenal and every inject type carries one or more domains, so every result can be read by domain. The question changes from "did this simulation pass?" to "how well does our email protection hold, across every simulation of the quarter?". This page lists the domains, explains where an inject gets its domains, and how the **Security Domains** widget scores them.

## From an action to a score per domain

::diagram{id=openaev-security-domains-classification}

## The ten domains

Every tenant starts with the same ten domains:

| Domain | The controls it tests |
| --- | --- |
| **Endpoint** | Protection of workstations and servers: EDR, antivirus, and the SIEM that receives their alerts |
| **Network** | Segmentation, firewalls, intrusion detection and prevention, traffic inspection |
| **Web App** | The security of your web applications: web application firewall, vulnerability detection |
| **E-mail Infiltration** | The protection of email flows: phishing detection, attachment filtering, anti-spam and anti-spoofing. It also holds the injects that reach people by email, SMS or media |
| **Data Exfiltration** | The detection and blocking of data leaving your network: data loss prevention, SIEM |
| **URL Filtering** | Web access control: proxies, URL categorization, zero trust network access |
| **Cloud** | Cloud-native controls: identity and access management, cloud logs in the SIEM |
| **Artificial Intelligence** | The AI systems you test, such as language models and AI agents, and their protections: LLM firewalls, AI gateways |
| **Tabletop** | Organizational exercises: decisions, procedures and coordinated response of your teams |
| **To classify** | Nothing tested yet: the action or the inject type declares no domain |

The names and colors of the ten domains are fixed. An integration can bring a new domain: it is created the first time an injector or a collector declares it, and then appears beside the ten.

## Where an inject gets its domains

An inject takes the domains of what it runs:

- **An action of the Threat Arsenal** carries the domains set in its **Domains** field, which you choose when you create or edit the action. See [Set the properties of an action](/openaev/use/threat-arsenal/action-properties/).
- **Another inject type** carries the domains its injector declares. The built-in email, SMS, media pressure, challenge and manual action injects carry **E-mail Infiltration** and **Tabletop**.

When an injector or a collector registers again, for example after an upgrade, OpenAEV adds the domains it declares to those already on its inject types. It never removes a domain you added by hand. A domain spelled differently, such as `Table-Top` for **Tabletop**, is matched to the existing one: letter case, spaces and punctuation do not count. **To classify** goes away as soon as a real domain is set.

Because the domain comes from the action, an expectation counts in each domain of its action, whatever the scenario it belongs to: a PowerShell command set to **Endpoint** scores the endpoint domain in every scenario that runs it.

## Read your results by domain

Domains appear wherever you filter or count injects: the **Domains** filter of the Threat Arsenal and of the inject type picker, the inject results, and the dashboards.

The **Security Domains** widget, shown on the home dashboard as **Performance by security domain**, has one card per domain. Each card has a line per expectation type, such as prevention, detection and vulnerability, with the share of those expectations that succeeded:

| Success rate | Color of the line |
| --- | --- |
| 25 % or less | Failure |
| 26 to 50 % | Warning |
| 51 to 75 % | Intermediate |
| More than 75 % | Success |

The domain's own icon takes the color of its worst line: a domain whose prevention fails shows as failing, however well detection does. A domain with no result in the period says so, and a click on a card opens the expectations behind it. To add the widget to your own dashboard, see [Create the widgets of a dashboard](/openaev/use/dashboards/widgets/).

:::example[Find the weak domain of Grand Meridian Hotels]
Over the first quarter of 2026, Grand Meridian Hotels replays Velvet Harbor every week against its front desks. **Performance by security domain** on the home dashboard shows:

- **Endpoint**: detection 82 %, prevention 64 %. The icon shows as intermediate: Meridian EDR sees the PowerShell loader but does not always block it.
- **E-mail Infiltration**: prevention 40 %. The icon shows as a warning: the Meridian email gateway lets the fake booking request for the European Hospitality Summit 2026 through more often than it blocks it.
- **Data Exfiltration**: no result, because no inject of the scenario exports data over the network yet.

The CISO, Hélène Marchetti, puts the email gateway first on the remediation plan, and asks for a data exfiltration step in the scenario.
:::

## What's next

- [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/): the actions and the domains they carry.
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/): what the success rates count.
- [Create the widgets of a dashboard](/openaev/use/dashboards/widgets/): add the **Security Domains** widget.
