---
title: Set the properties of an action
description: Every field of an action of the Threat Arsenal, tab by tab, from its name and domains to its command, its arguments, its prerequisites and its detection rules.
doc_type: reference
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/build/threat-arsenals/action-properties/
---

This page documents every field of an action of the [Threat Arsenal](/openaev/use/threat-arsenal/), tab by tab. Some fields are common to every action; others depend on its type. Creating an action is in [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/).

## The tabs of an action

::diagram{id=openaev-action-properties-tabs}

The form has four tabs, **General**, **Commands**, **Output** and **Remediation**. The **Output** and **Remediation** tabs are optional, and **Remediation** requires the Enterprise Edition.

## General properties

Set in the **General** tab.

| Property | What it is |
| --- | --- |
| **Name** | The name of the action |
| **Description** | What the action does |
| **Attack patterns** | The MITRE ATT&CK techniques it covers |
| **Tags** | Free-form labels to find and group it |
| **Domains** | The security control domains it belongs to, such as Endpoint or Network |
| **Expectations** | What the action is expected to raise: **Prevention**, **Detection** or **Vulnerability** |
| **Expected security platforms** | For each expectation, the security platform types expected to fulfil it; empty means any |

::screenshot{id=openaev-action-properties-general alt="The General tab of the action Velvet Harbor - booking web front discovery: its name, description, the attack patterns T1552.001 and T1046, the tag velvet-harbor, the Endpoint domain, the Prevention and Detection expectations, each expected from any security platform."}

## Commands properties

Set in the **Commands** tab. These apply to every type.

| Property | What it is |
| --- | --- |
| **Type** | **Command Line**, **Executable**, **File Drop** or **DNS Resolution** |
| **Architecture** | Where the command runs: **x86_64**, **arm64** or **All architectures** |
| **Platforms** | The operating systems it supports, such as Windows, Linux or macOS |
| **Prerequisites** | Commands run beforehand to make the action able to execute |
| **Cleanup executor** | The executor of the cleanup command |
| **Cleanup command** | The command that undoes what the action changed |
| **Arguments** | Values the command references at execution time |

::screenshot{id=openaev-action-properties-commands alt="The Commands tab of the same action: type Command Line, all architectures, Linux, the Sh attack command, then the Arguments, Prerequisites and Cleanup command sections."}

### Arguments

An argument sets a value in the command, the cleanup command or the prerequisites. There are two kinds:

- **Text argument**: a **Key**, used as a `#{key}` placeholder in the command, and a **Default value** that replaces it at execution and can be overridden when you create an inject. Other primitive types, such as **Port** or **Username**, work the same way and carry their own validation.
- **Targeted asset argument**: a **Key**, a **Targeted property** (**Hostname**, **Local IP (first)** or **Seen IP**) and a **Separator** that joins several values. At execution, OpenAEV fills it from each targeted asset.

:::info
To scan with `nuclei`, write the command `nuclei -t #{asset-key}` and add a targeted asset argument with the key `asset-key`. In the inject, set a source asset where `nuclei` is installed and the targeted assets to scan.
:::

### Prerequisites

| Property | What it is |
| --- | --- |
| **Command executor** | The executor of the prerequisite command |
| **Check command** | Verifies that a condition is met |
| **Get command** | Runs when the check command fails |

## Properties by type

| Type | Property | What it is |
| --- | --- | --- |
| **Command Line** | **Command executor** | The executor that runs the command |
| | **Command** | The command to execute |
| **Executable** | **Executable file** | The binary to run as its own process |
| **File Drop** | **File to drop** | The file placed on the host without running it |
| **DNS Resolution** | **Hostnames** | The host names to resolve to IP addresses |

## Output properties

Set in the **Output** tab: the rules that pull structured values from the raw output and, optionally, generate findings. The fields of a rule are in [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/).

## Detection remediation properties

Set in the **Remediation** tab, which shows a sub-tab for each security platform integrated in the platform. A rule describes how that platform detects the action.

:::tip[Enterprise Edition]
The **Remediation** tab requires a valid Enterprise Edition license. The AI generation of rules works for **Command Line** and **DNS Resolution** actions, and for CrowdStrike and Splunk security platforms.
:::

Each rule carries a status:

| Status | What it means |
| --- | --- |
| **Rules written by Human** | A person wrote the rules |
| **Rules generated by AI** | AI generated the rules |
| **Payload changed since rule was edited** | The action changed after the last AI generation; see the fields below |

::screenshot{id=openaev-action-properties-remediation alt="The Remediation tab of the action Velvet Harbor - booking database export: a sub-tab for Meridian EDR and one for Meridian email gateway, the rule of Meridian EDR marked Rules written by Human in its editor, and the Generate with AI button greyed out."}

The AI generation reads these fields: **Name**, **Description** and **Attack patterns** of the **General** tab, and **Type**, **Architecture**, **Platforms**, the command's executor and content, **Arguments** and the DNS host name of the **Commands** tab. Changing any of them marks an AI rule as outdated.

## What's next

- [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/): create, use and manage actions.
- [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/): the fields of the **Output** tab.
- [Security control domains](/openaev/concepts/security-domains/): how domains classify actions.
