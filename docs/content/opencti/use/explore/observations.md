---
title: Explore observations
description: Read what OpenCTI knows about an IP address, a domain or a hash, turn it into a detection rule, record a sighting, and handle malware samples safely.
sidebar_position: 7
doc_type: how-to
products:
  - opencti
personas: [soc-analyst, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-observations/
---

The page of an IP address, a domain, a URL or a hash tells you in one screen how dangerous the sources think it is, which threat and infrastructure it belongs to, which detection rules exist for it and which reports mention it. The fastest way to that page is to paste the value into the search bar: see [Search for knowledge](/opencti/use/explore/search/#check-one-value). This page shows how to read it, turn the value into a detection rule, record that your tools saw it, download a malware sample safely, and browse the **Observations** menu. The tabs every page shares are described in [Explore knowledge](/opencti/use/explore/#read-an-entity-page).

## Before you start

- Your role has the **Access knowledge** capability.
- Creating observables, indicators and sightings needs **Create / Update knowledge**. Deleting them needs **Delete knowledge**.

## Read what OpenCTI knows about a value

:::steps
1. Paste the value into the search bar and open the observable, or open :menu[Observations > Observables], filter on **Entity type** and search the value.
2. On **Overview**, read **Score** in **Basic information**: from 0 to 100, how malicious the sources consider the value. A value with no score and no link to a threat is not necessarily malicious: it can be one of your own servers.
3. Read **Indicators composed with this observable**: the detection rules built on this value. When the list is empty, nothing is ready to push to your detection tools for it yet.
4. Read **Latest created relationships**: the threats, infrastructures and other values it is linked to, such as the IP address a domain resolves to. **Latest containers about the object** lists the last reports, cases and observed data that hold it.
5. For the full picture, open **Knowledge**: **Nested objects** lists the objects inside the value, such as the sender of an email message, and **Relations** lists every relationship with its **Confidence level**. **Analyses** lists every report and case that contains it, and **Sightings** where your security tools saw it.
:::

::screenshot{id=opencti-observations-observable-overview alt="The Overview of an IPv4 address: its value and score, the indicators composed with this observable, and its latest relationships with an intrusion set and an infrastructure."}

:::only{opencti=">=7.261127"}
A **Software** observable shows, besides its name, vendor and version, its **PURL** in **Details**: the package URL of the software, such as `pkg:npm/hotel-booking-widget@2.4.1` for a package of the npm registry. Copy it to look the package up in your software inventory. You fill it in the creation and edition forms of the observable, and STIX 2.1 exports carry it as `purl` in the OpenCTI extension of the observable.

::screenshot{id=opencti-observations-software-purl alt="The Details of the Software observable hotel-booking-widget: its description, its vendor, the version 2.4.1 and the PURL pkg:npm/hotel-booking-widget@2.4.1."}
:::

Files, domains, URLs, hostnames, artifacts and network traffic also show **Malware analyses**: the results of sandboxes that analyzed them. To add context that the platform does not have yet, click **Enrichment** in the header and pick an enrichment connector: see [Enrich knowledge from external sources](/opencti/use/enrich/connectors/).

:::example[Qualify a domain from a phishing alert]
The mail gateway of a member hotel quarantined a booking request with a link to `booking-velvetharbor.example`. Paste the domain into the search bar and open the **Domain name**.

**Score** reads 90 / 100 and the label is `phishing`. **Indicators composed with this observable** lists the indicator `booking-velvetharbor.example`, so your detection tools can already receive it. **Latest created relationships** shows that the domain is related to **Velvet Harbor**, resolves to `203.0.113.10` and is part of **Velvet Harbor C2**. **Latest containers about the object** lists the report **Look-alike booking domains registered in bulk** and the incident response **Sighting of booking-velvetharbor.example**: another hotel saw the same domain on January 9, 2026, so add your alert to that case rather than opening a new one.
:::

The score of an observable is what its sources set. The score that decreases over time is the score of indicators: see [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/).

## Turn a value into a detection rule

When a value has no indicator yet, create one from its page.

:::steps
1. On the **Overview** of the observable, click **Add or create indicators**, the **+** next to **Indicators composed with this observable**.
2. Click **Create**, then confirm. OpenCTI creates a STIX indicator whose pattern matches the value, based on the observable. Click **Add** instead to link an indicator that already exists.
:::

The new indicator follows the decay rules of your platform and reaches the detection tools that read your [feeds](/opencti/use/share/feeds/). When you create an observable, the **Create an indicator from this observable** option of the creation form does the same in one step.

## Record that your tools saw it

A sighting records that a value or an indicator was seen in a place you protect, with how many times and when. It is how OpenCTI answers "did this hit us?" later.

:::steps
1. On the observable or the indicator, open **Sightings** and click **Create Sighting**. The **Create a sighting** panel opens.
2. Pick where it was seen: an organization, a sector, a location, a system or an individual.
3. Fill **Count**, the number of times it was seen, and the **First seen** and **Last seen** dates. Leave **False positive** off unless the activity turned out to be legitimate.
4. Click **Create**. The sighting shows in :menu[Events > Sightings] and on the page of the place it was seen.
:::

Sightings of an indicator are usually created by your SIEM or EDR through a connector. [Explore events](/opencti/use/explore/events/#sightings) explains how to read them, and a sighting can open a case on its own: see [Handle incidents and requests in cases](/opencti/use/analyze/cases/#when-a-sighting-opens-the-case).

## Download a malware sample safely

An artifact is an observable that holds a file, such as a malware sample. Its file is on **Overview**, under **Details**, with its MD5, SHA-1, SHA-256 and SHA-512 hashes.

:::steps
1. Open :menu[Observations > Artifacts] and click the artifact.
2. On **Overview**, click **Download**.
3. Choose **Encrypted archive** to get the file inside an AES-256 encrypted ZIP, or **Raw file** to get it as it is.
:::

:::warning
A sample can harm the computer that opens it. Prefer **Encrypted archive**. The archive password is `infected` unless an administrator changed it with `app:artifact_zip_password` (see the [configuration reference](/reference/opencti/configuration/#functional-customization)). Ask your administrator if `infected` does not open it.
:::

:::only{opencti=">=7.261119"}
The file is also listed under **Uploaded files** in the **Data** tab of the artifact. A click on its line does nothing there, and OpenCTI never opens it in the browser: **Download this file** and **Open in a new tab** both make you choose between **Encrypted archive** and **Raw file**. See [Data](/opencti/use/explore/#data).
:::

To add a sample, click **Create Artifact** in :menu[Observations > Artifacts] and upload the file. OpenCTI computes its hashes. A ZIP archive is unpacked and its first file becomes the artifact, so you can upload a sample in an archive protected with the platform archive password; Office documents are kept as they are.

:::only{opencti=">=7.261030"}
An archive whose content exceeds 100 MB is not unpacked: it is imported as is. Administrators change this limit with `app:artifact_zip_max_extracted_size` (`APP__ARTIFACT_ZIP_MAX_EXTRACTED_SIZE`, in bytes): see the [configuration reference](/reference/opencti/configuration/#functional-customization).
:::

:::only{opencti=">=7.260811"}
Instead of a file, **Create an artifact** accepts the **URL** where the sample is hosted. The artifact then records the address, without downloading the file.
:::

:::example[Add the attachment of a fake booking request]
The mail gateway of a member hotel extracted the attachment of a fake booking request, `booking_request.zip`, protected with the password `infected`. In :menu[Observations > Artifacts], click **Create Artifact**, upload the archive, write in **Description** that it came from a booking request received by Grand Meridian Hotels, and click **Create**.

The artifact holds the file inside the archive, with its four hashes. Search the SHA-256 hash in the search bar to see whether a file of the same hash is already known, for example as the sample of Velvet Loader.
:::

## Browse the Observations menu

| Entry | What it holds | What to look at |
| --- | --- | --- |
| **Observables** | Technical values seen on networks and systems: IP addresses, domain names, URLs, email addresses, file hashes and more | **Score**, the indicators and threats it is linked to |
| **Artifacts** | Observables that hold a file, such as a malware sample | The file, its hashes and its malware analyses |
| **Indicators** | Detection rules, written as a pattern in STIX, Sigma, YARA or another language, valid for a period of time | **Valid until**, **Score** and whether it is used for **Detection** |
| **Infrastructures** | Systems and services used for a purpose, such as the command and control servers of an attack | The observables it consists of and the threats that use it |

An indicator is based on observables, and both link to the threats they indicate: this is where indicators of compromise meet the threat context. See [Data model](/opencti/concepts/data-model/) for the difference between an observable and an indicator.

**Indicators.** An indicator page shows its **Indicator pattern**, its **Valid from** and **Valid until** dates, its **Score**, whether it is used for **Detection**, its **Main observable type** and the observables it is based on. Its **Sightings** tab shows where it was seen. How its score decays and when it expires is in [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/). To send indicators to your SIEM, see [Feeds](/opencti/use/share/feeds/).

When you create an indicator with **Create Indicator**, **Indicator types** says what a match means, from the indicator type vocabulary: `malicious-activity`, `anomalous-activity`, `compromised`, `anonymization`, `attribution`, `benign` or `unknown`. Your detection team reads it to decide how to treat an alert. **Create observables from this indicator**, off by default, reads the STIX pattern and creates each observable it names, linked to the indicator by `based-on`, with the score, author, markings and labels of the indicator. Turn it on when you want the values themselves in your knowledge, for example to find them in search and link them to an infrastructure.

:::only{opencti=">=7.261127"}
Only an equality names an observable: a value the pattern excludes, such as `[ipv4-addr:value != '203.0.113.10']`, creates none. A quote escaped in the pattern is kept in the value: `[url:value = 'https://booking-velvetharbor.example/?ref=\'vh\'']` creates the URL `https://booking-velvetharbor.example/?ref='vh'`.
:::

:::example[List the Velvet Harbor indicators that expire within 30 days]
Before the indicators of the hospitality campaign leave your detection tools, you want to review them and extend those that are still active. In :menu[Observations > Indicators], add two filters:

- **In regards of**, operator **Equals**, **Relationship type** `indicates`, **Entity** **Velvet Harbor**: the indicators that indicate the intrusion set.
- **Valid until**, operator **Within**, **From** `now` **To** `now+30d`.

The list shows the indicators `203.0.113.10` and `booking-velvetharbor.example` when their validity ends in the next 30 days. Keep the relative dates: saved as a filter, the list stays current every time you open it.
:::

**Infrastructures.** An infrastructure page shows its **Infrastructure types**, its **First seen** and **Last seen** dates and an **Observables distribution** chart. Its **Knowledge** tab opens on the diamond model, a timeline and the global kill chain, to see which threats use it and what they target. Its **Observables** section lists the addresses and domains it consists of, communicates with or is related to, the list to block when a command and control infrastructure is confirmed.

:::only{opencti=">=7.261127"}
The number beside **Observables** counts the observables of that list, each one once.
:::

## Create several observables at once

When you create an observable, types that one value identifies, such as a domain name, an IP address, a URL or a file hash, accept a list: open **Create multiple observables** in the creation form and paste one value per line. For files, choose whether your list holds names or MD5, SHA1, SHA256 or SHA512 hashes. Types identified by several fields, such as Software (name, vendor, version, CPE), do not accept a list, and the form says so. To create many observables of other types, import a CSV file with a [CSV mapper](/opencti/configure/mappers/csv/). See [Create knowledge manually](/opencti/use/ingest/manual/) for the creation form.

## What's next

- [Search for knowledge](/opencti/use/explore/search/) to check a value or a whole list in seconds.
- [Explore events](/opencti/use/explore/events/) for the sightings of your indicators.
- [Handle incidents and requests in cases](/opencti/use/analyze/cases/) to open a case when a value hit your systems.
- [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/) for scores, decay and expiration.
- The [data model reference](/reference/opencti/data-model/#observations) for the attributes and relationships of each type of the Observations menu, and of every observable type in its Observables section.
