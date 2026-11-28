---
title: Search for knowledge
description: Check in seconds whether OpenCTI knows an IP address, a domain or a hash, check a whole list at once, or search by keyword, by question or inside files.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [soc-analyst, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/search/
---

To check an IP address, a domain, a URL or a hash from an alert, paste it into the search bar at the top of any OpenCTI screen and press Enter. In a second you see whether the platform knows the value, as an observable, an indicator or both, who reported it and in how many reports it appears. A whole list of values goes to [bulk search](#check-a-list-of-values). To find knowledge by name, by a question in plain language or inside uploaded documents, read on.

## Before you start

- Your role has the **Access knowledge** capability.
- **Ask AI** and **Files search** need an Enterprise Edition license. **Ask AI** also needs generative AI configured on the platform, and **Files search** needs [file indexing](/opencti/configure/file-indexing/) started.

:::only{opencti="<7.261127"}
Without **Access knowledge**, the search bar does not show.
:::

:::only{opencti=">=7.261127"}
Without **Access knowledge**, the search bar only finds the pages of the platform you can open: see [Open a page of the platform](#open-a-page-of-the-platform).
:::

## Check one value

::::steps
1. Click the search bar in the top bar and paste the value as you received it. Defanged values such as `hxxp://` or `example[.]com` are refanged before the search.

   :::only{opencti="<7.261127"}
   The search bar reads **Search the platform...**.
   :::

   :::only{opencti=">=7.261127"}
   The search bar reads **Type / to search**: press `/` on any screen to put the cursor in it, unless you are typing in another field or a dialog is open. To pick another key, or to turn the shortcut off so that the bar reads **Search the platform...**, see [Choose the key that opens the search](/opencti/use/profile/#choose-the-key-that-opens-the-search).
   :::

2. Press Enter. The **Knowledge search** tab lists the objects that match, the best matches first.
3. Read each line: **Type** says what the value is in the platform, **Author** who reported it, **Labels** how it was tagged, and **Analyses** in how many reports and cases it appears.
4. Click the observable or the indicator. Its **Overview** gives the score, the threats it relates to and the latest reports about it: [Explore observations](/opencti/use/explore/observations/#read-what-opencti-knows-about-a-value) explains how to read it.
::::

::screenshot{id=opencti-search-keyword-results alt="Knowledge search results for 203.0.113.10: an indicator and an IPv4 address of that value, both by Example CERT and labeled c2, then two reports that mention the address."}

:::example[Check whether 203.0.113.10 belongs to Velvet Harbor]
The EDR of a hotel front desk flags a connection to `203.0.113.10`. Paste the address into the search bar and press Enter.

Four lines come back: the **Indicator** `203.0.113.10` and the **IPv4 address** `203.0.113.10`, both by Example CERT and labeled `c2`, the report **Velvet Harbor targets hotel booking systems**, whose text mentions the address, and the incident report of Grand Meridian Hotels, where its EDR saw the address on the booking servers. Open the IPv4 address: **Score** reads 80 / 100, and **Latest created relationships** shows it is related to the intrusion set **Velvet Harbor** and part of the infrastructure **Velvet Harbor C2**. The front desk talked to Velvet Harbor's command and control: [open a case](/opencti/use/analyze/cases/#open-a-case) from there.
:::

The keyword search also matches words that start with what you typed, and text inside descriptions. To know whether this exact value is known, and nothing else, use bulk search: it matches whole values only.

### Search from your other tools

Each search has its own address, so a browser shortcut or a link in your SIEM opens the result directly:

- `https://opencti.example.com/dashboard/search/knowledge/203.0.113.10` runs a keyword search for `203.0.113.10`.

:::only{opencti=">=7.260811"}
- `https://opencti.example.com/dashboard/search_bulk?q=203.0.113.10,198.51.100.7` opens bulk search with these values. Separate them with commas, semicolons or line breaks (`%0A`).
:::

:::example[Search OpenCTI from the browser address bar]
Your team checks dozens of values a day. In your browser settings, add a site search with the shortcut `octi` and the address `https://opencti.example.com/dashboard/search/knowledge/%s`. Typing `octi booking-velvetharbor.example` in the address bar now opens the OpenCTI results for the domain, without going through the search bar.
:::

## Check a list of values

Bulk search checks a whole list at once, such as the addresses of a firewall alert or the CVEs of a bulletin, and splits it into what the platform knows and what it does not.

:::steps
1. In the search bar, click **Bulk search**, paste your values and press Enter. The bulk search page opens with your values in **Keywords to search**, one per line.
2. Add or change values in **Keywords to search**, one per line or separated by commas. The results update as you type.
3. Read **Known entities**: the objects that match a value, with the number of **items** on the tab. Export them, or select them to act on them as in any list.
4. Open **Unknown entities** to see the values the platform does not know.
5. To record some of them, select them and click **Create unknown entities** in the toolbar. The **Import data** panel opens with a file of the selected values, which you can import directly or into a draft. See [Import files](/opencti/use/ingest/files/).
:::

::screenshot{id=opencti-search-bulk alt="Bulk search with five pasted values: the Known entities tab lists the three objects the platform knows, and the Unknown entities tab counts the two others."}

:::example[Check three IP addresses from a firewall alert]
The firewall of a member hotel blocked three addresses overnight. Click **Bulk search** and paste:

```text
203.0.113.10
198.51.100.7
192.0.2.200
```

**Known entities** shows three lines: the IPv4 address `203.0.113.10` and the indicator of the same name, both tied to the Velvet Harbor command and control, and the IPv4 address `198.51.100.7`, the proxy of the fake booking pages. **Unknown entities** holds `192.0.2.200`: select it and click **Create unknown entities** to record it, so that the next alert on this address finds it.
:::

Bulk search looks for a case-insensitive exact match, not a partial one, in these attributes: name, aliases, MITRE ID, value, subject, abstract, additional names, hashes (MD5, SHA-1, SHA-256, SHA-512), URL, path, IBAN, card number, account login and type, user ID, persona name and attribute key.

## Find knowledge by keyword

The same search bar finds threats, reports and any other object by name. It searches all STIX domain objects and STIX cyber observables you can access. Results are sorted by relevance: matches in the name, aliases and identifiers come first, then in the description, then in other attributes. Narrow them with filters, for example on the entity type or the author, as in any list. Pasting an internal or STIX identifier finds its object.

Every word you type gets an implicit trailing wildcard: `apt2` also finds `apt28` and `apt29`. You can also use:

| Syntax | Effect | Example |
| --- | --- | --- |
| Quotes `"…"` | Matches the exact phrase, not each word | `"lazarus group"` |

:::only{opencti=">=7.260811"}
A `*` inside a word matches any characters at that place: `ap*9` finds `apt29`.
:::

:::only{opencti=">=7.261030"}
A `~` at the end of a word matches close spellings, with an optional edit distance of 0, 1 or 2: `lazarous~` finds `lazarus`, `lazarous~1` allows one edit. A fuzzy word gets no implicit wildcard.
:::

:::example[Find every spelling of Velvet Harbor]
Three sources reported the same intrusion set under slightly different names, and you want to find them all before merging them.

1. In the search bar, type `velvet harb` and press Enter. With the implicit wildcard, `harb` matches both `harbor` and `harbour`.
2. On **Knowledge search**, add the filter **Entity type**, operator **Equals**, value **Intrusion set**.

The list keeps three intrusion sets: **Velvet Harbor** by Example CERT, **Velvet Harbour** by Saltmarsh CERT and **Velvet Harbor Group** by Bluefin Labs. Without the filter, the reports, malware, indicators and infrastructure that mention the name follow them. [Merge them](/opencti/use/enrich/merge/) to keep one.
:::

::::only{opencti=">=7.261127"}
## Open a page of the platform

The search bar also finds the pages of the platform, such as a list, a tab or a settings page, so you reach them without going through the menus.

:::steps
1. In the search bar, type a few words of the page you want. From the second character, a list opens under the field.
2. Each word you type must start a word of the page name, of its place in the menus or of the keywords the page is known by, in any order. Case and accents do not matter, and English words work whatever your language.
3. Read the list. Its first line, **Search** followed by what you typed, runs a knowledge search. Up to ten pages follow, those whose name matches first. Each page shows its place in the menus, such as :menu[Data > Data sharing], and between quotes the keyword that matched when its name and place do not show it.
4. Click a page, or highlight it with the arrow keys and press Enter: the page opens and the search bar empties. Ctrl-click or Command-click opens the page in a new tab, and Escape closes the list.
:::

::screenshot{id=opencti-search-application-pages alt="The top bar search with blocklist typed: under the field, the Search: blocklist line, then the pages Indicators, under Observations, and CSV feeds, under Data > Data sharing, each with the keyword blocklist in quotes."}

The list only holds the pages you can open, as your capabilities, the license and the current draft allow. Without the **Access knowledge** capability, the list has no **Search** line, and Enter does nothing until you highlight a page.

When you press Enter on a keyword instead, the **Knowledge search** tab shows the matching pages in an **Application pages** card above the results: two pages first, then up to ten with **Show more**.

:::example[Find where the blocklist is published]
A member hotel asks which feed carries your blocklist, and you no longer remember the menu of the feeds. Type `blocklist` in the search bar. Under the **Search** line, the list shows **Indicators**, under :menu[Observations], and **CSV feeds**, under :menu[Data > Data sharing], both with the keyword “blocklist”. Click **CSV feeds**: the list of the CSV feeds the platform publishes opens, and the search bar is empty again.
:::
::::

## Ask a question in plain language

**Ask AI** turns a question into filters. For example, *intrusion sets targeting the hospitality sector in Europe* becomes filters on the type, the targeted sector and the targeted region.

:::steps
1. In the search bar, click **Ask AI**. If the Filigran AI Terms have not been accepted yet, the platform asks an administrator to validate them first.
2. Type your question or statement and press Enter.
3. Check the filters the model built above the results, and adjust them if needed. A message says when some entities you mentioned were not found, or when no filter matched the question.
:::

The model builds one level of filters joined by a single and/or mode, so check them before you trust the list. A question outside the cyber threat context may return no filters. With your own model endpoint, each question sends a large prompt, which costs tokens. See [Refine content with AI](/opencti/use/enrich/refine-content/) for the AI setup.

::::only{opencti=">=7.260811"}
:::xtm-one[Natural language search with XTM One]
When XTM One is connected to the platform, an XTM One agent can translate your question into filters instead of the platform model. If no agent is available for this task, the search says so. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

:::only{opencti=">=7.261030"}
To choose which XTM One agent translates your question, click **Choose AI agent**, the arrow next to **Ask AI**.
:::

## Search inside files

**Files search** extends a keyword search to the text of the files uploaded to the platform, in :menu[Data > Import] or in the **Data** tab of an object. It finds what was never turned into structured knowledge, for example a hostname that appears only in the PDF of a vendor report.

:::steps
1. Search a keyword from the search bar.
2. Open the **Files search** tab, next to **Knowledge search**. Its counter shows how many files match.
3. Open a file, or the object it is attached to in a new tab.
:::

If the tab says **File indexing is not started.**, indexing must be set up first. Users with the **Access to file indexing** capability see a **Configure file indexing** button that leads to its settings; others are asked to contact their administrator. See [Search inside files](/opencti/configure/file-indexing/).

:::only{opencti=">=7.261127"}
In a draft, where the settings do not open, the button reads **Exit draft and configure file indexing**: it leaves the draft, then opens the file indexing settings.
:::

## Search within a list

Every list has its own search box above the lines. It searches only the objects of that list, combines with the list filters, and applies when you export the list. Graphs have a search box too, to find a node: see [Work with graphs](/opencti/use/analyze/graphs/).

:::only{opencti=">=7.261127"}
When you type in the value field of a filter, such as **Author** or **Labels**, the suggestions list the best matches first, as the keyword search ranks them, grouped by entity type. Before you type, they stay in alphabetical order.
:::

To search all entities or all relationships with filters, without starting from one menu, open :menu[Data > Entities] or :menu[Data > Relationships]. These two lists cover every type and accept the same filters as the other lists. The [filter key reference](/reference/opencti/filter-keys/) lists every filter key and operator, including the special keys of the filter menu such as **In regards of**.

## What's next

- [Explore observations](/opencti/use/explore/observations/) to read the page of the observable or indicator you found.
- [Pivot and investigate](/opencti/use/analyze/pivot/) from a search result to the objects around it.
- [Handle incidents and requests in cases](/opencti/use/analyze/cases/) to open a case when the value is malicious.
- The [filter key reference](/reference/opencti/filter-keys/), every key and operator you can filter on.
