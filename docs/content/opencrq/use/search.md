---
title: Find a scenario, an asset or a threat by its name
description: Search the risk scenarios, primary assets, supporting assets, threats, risk scopes and controls of a tenant by their name from the top bar, and open the one you need.
sidebar_position: 8
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, executive]
owner: docs-opencrq
last_reviewed: 2026-11-23
---

The **Search the platform...** field, at the top of every page of a tenant, finds a risk scenario, a primary asset, a supporting asset, a threat, a risk scope or a control by its name. You open the scenario a colleague named in a meeting, or the server a finding mentions, without knowing which list holds it. The results come grouped by type, and the address keeps the search, to bookmark or send to a colleague.

## Before you start

Every member of a tenant can search it: no capability is needed. The search covers the whole tenant you are in, whichever risk scope is selected, and finds only what you may read:

- an asset or a threat that carries a marking you are not cleared for is not among the results, as in its own list: see [Limit what people see with markings](/opencrq/administer/users-and-roles/#limit-what-people-see-with-markings);
- a threat from OpenCTI shows only when OpenCTI lets you read it: see [Which threats you see](/opencrq/use/threats/threat-center/#which-threats-you-see).

The field is not on the pages of the platform administration, which belong to no tenant.

## Search the tenant

:::steps
1. Click **Search the platform...** at the top of the screen.
2. Type part of the name you are looking for, and press Enter. The **Search** page opens, titled with your search, such as "Results for “booking”".
3. Click a match to open it, or **View all** on a card to see more matches of its type.
:::

::screenshot{id=opencrq-search alt="The Search page for booking at Grand Meridian Hotels: the Risk Scenarios card with 1 match, Ransomware on the Booking Platform, the Primary Assets card with 1, Online booking platform, and the Assets card with 2, Booking database server and booking.grand-meridian.example."}

Each card holds one type, with its number of matches beside its title, and lists its first ten matches in alphabetical order, controls in the order of their catalogue. A type with no match has no card. **View all** shows when a type has more than ten matches. The cross in the field, **Clear search**, empties it.

| Card | A match opens | **View all** opens |
| --- | --- | --- |
| **Risk Scenarios** | The scenario | The list of risk scenarios, without the search |
| **Primary Assets** | The primary asset | The list of primary assets, filtered on your search |
| **Assets** | The supporting asset | The **Assets** list, filtered on your search |
| **Threats** | The threat, in the threat center | The threat center, filtered on your search |
| **Risk scopes** | The risk scope, on its home dashboard | The list of risk scopes, without the search |
| **Controls** | The **Controls** list, filtered on the control's name | The **Controls** list, filtered on your search |

The address of the page holds the search, such as `/search?search=booking` after the tenant's id. Whoever opens it sees only the matches they may read.

## How a name matches

- A scenario, a primary asset, an asset, a risk scope or a control matches when its name contains your text, wherever it stands. Case does not matter.
- Accents do not matter either for scenarios, primary assets and assets: `hotel` finds a name that holds "hôtel". For risk scopes and controls, type the accents as the name holds them.
- A threat matches when its name starts with your text, as in the threat center's own search: `ember` finds Ember Tide, `tide` does not.
- An asset matches by the name the **Assets** list shows: its own name, or the one its source gave it when it has none.
- The search reads names only, not descriptions, and takes your text as one piece, spaces included: `booking platform` finds the Online booking platform, not the Booking database server. Spaces before and after the text do not count, and the text holds 200 characters at most.

## Check that it works

- Search for the exact name of a scenario you know: the **Risk Scenarios** card lists it.
- When nothing matches, the page says "Nothing matches this search", and offers **Open risk scenarios** and **Open assets** to browse the tenant's lists. Check the spelling, type a shorter part of the name, and for a threat, its first word.
- A threat or an asset you expect is missing while its name matches: it carries a marking you are not cleared for, or, for a threat from OpenCTI, OpenCTI does not let you read it. See [Before you start](#before-you-start).
- The page opened without a search, from its address, shows "Search the platform": type your search in the field at the top.

:::example[Find everything about the booking platform]
Claire Morel, the cyber risk manager of Grand Meridian Hotels, prepares the steering committee on the ransomware risk. She wants every object of the tenant that concerns the booking platform, in France as in Spain. She clicks **Search the platform...**, types `booking`, and presses Enter. The page "Results for “booking”" shows three cards:

| Card | Count | Matches |
| --- | --- | --- |
| **Risk Scenarios** | 1 | Ransomware on the Booking Platform |
| **Primary Assets** | 1 | Online booking platform |
| **Assets** | 2 | Booking database server, booking.grand-meridian.example |

No threat name starts with "booking", and no risk scope or control name contains it: there is no other card. She opens Ransomware on the Booking Platform to read its price, then comes back with the browser's back button and opens booking.grand-meridian.example, to show the committee the chain behind the platform on its **Business dependencies** tab.

She then types `booking platform`: the scenario and the primary asset remain, and the **Assets** card is gone, as neither asset's name holds both words in that order.
:::

## What's next

- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), to work on the scenario you found.
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/), to read an asset and its business dependencies.
- [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/), to filter the threats by more than their name.
