---
title: Investigate an incident
description: Work one night's intrusion as a case in OpenCTI, from the lure to the people it reached, replay it in OpenAEV, price it again in OpenCRQ and brief the CISO with XTM One.
doc_type: journey
journey: investigate-an-incident
products:
  - opencti
  - openaev
  - opencrq
  - xtm-one
personas: [soc-analyst, validation-engineer, risk-manager]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 61
---

At the end of this journey, an intrusion is understood and closed: you know what the lure posed as, what was seen and where, who it reached and which threat is behind it, the response is tracked as a case in OpenCTI, OpenAEV has replayed the attacker's techniques to check the fix, OpenCRQ has priced the breach again, and XTM One has summarized it for the CISO. An incident responder drives it, with the validation engineer and the risk manager. The steps below are the pages to follow, in order.

::diagram{id=journey-investigate-an-incident}

## Who takes part

- **SOC analysts**, as incident responders, read the incident, its sightings and the people it reached, then open and run the case (steps 1 to 4), and ask XTM One for the summary (step 7).
- **Security validation engineers** replay the attacker's techniques on the endpoints and read what the defenses stopped (step 5).
- **Risk managers** read the new price of the breach and keep it in the risk register (step 6).
- CTI analysts attribute the incident to its intrusion set in OpenCTI, and the CISO receives the summary.

## Why it spans several products

An investigation answers four questions, and each product answers one:

- **What happened?** OpenCTI holds the incident, the sightings of the attacker's indicators at the victim, the event the lure used and the people it targeted, next to the reports on the threat. The case gathers them with the tasks of the response, so the next analyst who opens the intrusion set finds it.
- **Would it work again?** OpenAEV replays the techniques of the threat on your own endpoints and scores what your security tools prevented and detected, once the response is done and on a schedule after.
- **What does it cost?** OpenCRQ prices the breach as an annualized loss, and prices it again as the controls change and the test results come in.
- **What do I tell the CISO?** XTM One reads the case and its knowledge with your rights and writes the readout.

Alone, the case says what happened on one night. With the replay and the price, it says whether the fix holds and what the risk is now. How a container holds its evidence is in [Containers](/opencti/concepts/containers/); why a test result moves a price is in [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## What you need

- **OpenCTI**, with the incident and the threat behind it: here, the incident reported by the victim and the reports of Example CERT on Velvet Harbor.
- **OpenAEV**, connected to OpenCTI so that the **OpenAEV Coverage** connector shows among its connectors, with OpenAEV agents on the endpoints to test and a collector that reads your EDR. See [Connect OpenAEV to OpenCTI to test security coverages](/openaev/use/coverage-enrichment/).
- **OpenCRQ**, with the risk scope of the victim, its risk scenario, and OpenAEV connected to the tenant. See [Import OpenAEV results](/opencrq/use/sources/openaev/).
- **XTM One**, with the platforms registered with it. **Ask Ariane** requires the Enterprise Edition, and an administrator who has accepted the Filigran AI Terms. See [Connect the modules to each other](/deploy/configure/connect-modules/).
- Capabilities:
  - in OpenCTI, **Access knowledge** to read, and **Create / Update knowledge** to attribute the incident, open the case and add its tasks; restricting the case to its participants needs the Enterprise Edition and **Manage authorized members**;
  - in OpenAEV, **Access assessment** to read the scenario, **Manage assessment** to change its injects, **Manage tenant settings** for the default asset rules, and **Manage threat arsenal** to add a payload;
  - in OpenCRQ, any member of the tenant works on risk scenarios.

## How it unfolds

The story: on the evening of 8 January 2026, Paul Garnier, the night auditor of Grand Meridian Hotels in Paris, opened a booking request for the European Hospitality Summit 2026. At 02:10, the shared `night.audit` front-desk account exported the guest records from the booking database server. Hélène Marchetti, the CISO, reported the incident at 08:00. Example CERT, which serves the hotel chains, works it in OpenCTI.

### Find what the lure posed as

The booking request named the European Hospitality Summit 2026, a trade fair of the hotel industry in Paris from 13 to 15 January. Its page in :menu[Entities > Events] shows the threats that target it, among them Velvet Harbor and Operation Night Desk, and its sighting at Grand Meridian Hotels: booking requests that named it reached the front desk 4 times, from 5 to 8 January ([step 1](/opencti/use/explore/entities/)).

### Read the incident and what was seen

In :menu[Events > Incidents], **Unauthorized access to booking database** is a `compromise` of `high` severity, first seen at 02:10 on 9 January, reported by Grand Meridian Hotels ([step 2](/opencti/use/explore/events/)). Its sightings tie it to the threat: the domain `booking-velvetharbor.example` was seen 14 times at the hotel chain between 21:40 on 8 January and 03:15, and `203.0.113.10`, Velvet Harbor's command and control server, on the booking database server from 02:10. The analyst attributes the incident to Velvet Harbor, with a confidence that rests on the method until more indicators are sighted.

### See who it reached

The page of Paul Garnier in :menu[Entities > Individuals] shows what targeted him: Velvet Harbor, Operation Night Desk, Velvet Loader and the spearphishing attachment technique, and the sighting of `booking-velvetharbor.example` recorded at him at 21:40 on 8 January, the evening he opened the request ([step 3](/opencti/use/explore/entities/)). The CISO, Hélène Marchetti, has her own page as the person who reported the incident.

### Work it as a case

The analyst on duty opens the incident response `Booking system intrusion at a hotel chain`, dated to the night of the export, `high` and `P2` ([step 4](/opencti/use/analyze/cases/)). The incident goes in **Origin of the case**, the two indicators in **Observables**, Velvet Harbor in **Entities**. **Correlated containers** shows that another member hotel saw the same domain the day before. The tasks split the response: collect the booking platform logs, reset the front-desk accounts, notify the payment card brands.

### Replay the attack to check the fix

Once the front-desk accounts are reset, the validation engineer asks for a weekly security coverage of Velvet Harbor in OpenCTI. OpenAEV builds the scenario `Coverage of Velvet Harbor`, with one inject per technique it has a payload for, such as PowerShell and Web Protocols, and a disabled placeholder for the spearphishing attachment until a payload is added ([step 5](/openaev/use/scenarios/from-opencti/)). Each simulation scores what Meridian EDR prevented and detected, and sends the scores back to the coverage in OpenCTI.

### Price the breach again

In OpenCRQ, the risk scenario `Data Breach on Guest Records` of Grand Meridian France was priced at $97K a year on 12 January ([step 6](/opencrq/use/quantify/risk-scenarios/)). OpenCRQ prices it again on its own when its findings, controls or threats change: its OpenAEV connector reads the finished simulations every hour by default. After multi-factor authentication reaches every front-desk account on 2 March, the next run falls to $33K, and its **Simulation History** keeps both.

### Summarize it for the CISO

**Ask Ariane** answers "Summarize for the CISO the case Booking system intrusion at a hotel chain: what happened on the night of 9 January, what we did, and what is still open". It reads the case, its tasks and the knowledge it holds with the rights of the person who asks ([step 7](/xtm-one/use/in-opencti/)). **AI Insights** on the case gives a summary of its content and activity. Check the names and dates of the answer against the case before it goes to the CISO.

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Check your defenses against the threats you track](/workflows/threat-informed-validation/), to keep testing the threat every week once the case is closed.
- [Run a CTEM cycle across the platform](/workflows/ctem-cycle/), to choose what to fund from the price of the breach.
- [Automate threat intelligence](/workflows/automate-intelligence/): a playbook can open a case with its tasks for every new incident.
- [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/), to see how a sighting of a valid indicator raises an incident on its own.
