---
title: Keep benign values out of indicators
description: Stop OpenCTI from creating indicators for benign values, such as your own IP ranges or popular domains, with exclusion lists that apply to every source.
sidebar_position: 7
doc_type: how-to
products:
  - opencti
personas: [admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/exclusion-lists/
---

An exclusion list holds values you know to be legitimate, such as your own address ranges, your domains or well-known services. OpenCTI refuses to create an indicator whose STIX pattern contains one of these values, whatever the source. This page shows how to create a list, check that every platform node uses it, and keep it up to date.

## Before you start

- Your role has the **Manage customization** capability.
- Your values are in a plain text file (`.txt`) with one value per line, or ready to paste. A file is limited to 10 MB by default (`app:exclusion_list:file_max_size`, see [Functional customization](/reference/opencti/configuration/#functional-customization) in the configuration reference).
- Each list holds one family of types: IP addresses (**IPv4 address**, **IPv6 address**), or the other observable types. A list cannot mix both.

## How values match

| Type | Write values as | Matches |
| --- | --- | --- |
| **IPv4 address**, **IPv6 address** | An address, `192.168.20.10`, or a CIDR range, `192.168.0.0/16` | The address, or every address in the range |
| **Domain name** | A name, `example.com`, or a name that starts with a dot, `.example.com` | The exact name. With a leading dot, every subdomain, such as `www.example.com` and `booking.eu.example.com`, but not `example.com` itself: list both values to cover the name and its subdomains |
| **URL** | A full URL, `https://www.example.com/login`, or its end from a dot, `.example.com` | The exact URL. With a leading dot, a URL that ends with that value: `.example.com` matches `https://www.example.com` but not `https://www.example.com/login` |
| **File**, **Artifact** | A hash, such as a SHA-256 | A pattern that contains one of the hashes in the list |
| **Hostname**, **Email address** | The value | The exact value |

Write values in the case your sources use, in practice lowercase for domains and hashes: a value in another case is not reliably matched.

## Create an exclusion list

:::steps

1. Open :menu[Settings > Customization > Exclusion lists] and click **Create Exclusion List**. The **Create an exclusion list** panel opens.
2. Enter a **Name** and a **Description** that says where the values come from.
3. In **Apply on indicator observable types**, choose the types the values belong to. Choosing an IP type with another type shows `Incompatible types, can't mix IP types with other types`.
4. Add the values. **Upload file** is on: select your `.txt` file under **Associated file**. To type the values instead, turn **Upload file** off and paste them in **Content (1 / line)**.
5. Click **Create**. The list appears in the table, active, with its types in the **Entity type** column.

:::

::screenshot{id=opencti-exclusion-list-create alt="The Create an exclusion list panel with the Name and Description fields, the observable types to apply the list on, and the Upload file switch on with its file selector."}

:::example[Never flag the member network as malicious]
The feeds Example CERT ingests sometimes report private addresses seen in sandbox runs, and a detection tool that receives `[ipv4-addr:value = '10.20.30.40']` would alert on the hotel members' own network.

- **Name**: `Internal ranges`
- **Description**: `Private address space of the member network`
- **Apply on indicator observable types**: **IPv4 address** and **IPv6 address**
- **Upload file**: on, with the file `internal-ranges.txt` in **Associated file**:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
fd00::/8
```

Once **Status** reads **Synchronized**, a feed that sends an indicator for `10.20.30.40` gets the error `Indicator of type stix is contained in exclusion list.` and nothing is created. The member domains go in a second list, `Trusted domains`, applied on `Domain name`, with `example.com` and `.example.com` to cover the domain and every subdomain; URLs on those domains need their own values, because a leading dot only matches a URL without a path. The file limit stays at its default; to accept larger files, raise `app:exclusion_list:file_max_size`:

```yaml
services:
  opencti:
    environment:
      APP__EXCLUSION_LIST__FILE_MAX_SIZE: 20000000
```
:::

## Check that the list is applied

OpenCTI copies the lists to a cache on every platform node. The cards above the table show where that stands:

- **Status** reads **In progress** while the nodes load the change, then **Synchronized**.
- **Last modification date** is the last change to any list; **Current cache version date** is the version every node uses. When both dates match, every node applies your list.

::screenshot{id=opencti-exclusion-lists-status alt="The Exclusion lists screen with the Status card showing Synchronized, identical Last modification date and Current cache version date, and a table of two active lists with their number of elements."}

The table also gives each list's **Number of elements**, **Creation date** and whether it is **Active**.

## Keep a list up to date

Open the row menu of a list:

- **Update** opens the **Update an exclusion list** panel to change the name, description, types or values. You can edit the values in place when the file is under 1 MB; for a larger list, turn on **Upload file** and send a new file, which replaces the old one.
- **Disable** pauses the list without deleting it; **Enable** turns it back on.
- **Download file** gives you the current values as a text file.
- **Delete** removes the list.

Every change starts a new synchronization: wait for **Synchronized** before you rely on it.

## What gets excluded

When a connector, a feed (CSV, JSON, RSS, TAXII), a playbook or a user creates an indicator, OpenCTI extracts the observable values from its STIX pattern. If one of them is in an active list, the indicator is not created and the source receives the error `Indicator of type stix is contained in exclusion list.`

:::note
Exclusion lists apply to indicators with a STIX pattern only. An observable created on its own, for example by a feed that does not create indicators, is not filtered. Indicators already in the platform stay as they are.
:::

## What's next

- [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/) to see how indicators are created, scored and decayed.
- [Keep indicators from decaying](/opencti/configure/decay/exclusions/) to keep indicators from decaying, rather than to block them.
