---
title: Check your defenses against the threats you track
description: Turn a threat known in OpenCTI into OpenAEV tests on your endpoints, read the scores back in OpenCTI, price what they missed in OpenCRQ, and sum it up with XTM One.
doc_type: journey
journey: threat-informed-validation
products:
  - opencti
  - openaev
  - opencrq
  - xtm-one
personas: [validation-engineer, soc-manager, cti-analyst]
owner: docs-openaev
last_reviewed: 2026-11-09
---

At the end of this journey, you know whether your defenses stop a threat your intelligence describes, technique by technique, and you keep knowing it every week. A threat known in OpenCTI becomes an OpenAEV scenario on the endpoints it targets, your security tools' alerts score each technique, the scores come back to OpenCTI as a security coverage, OpenCRQ prices the gaps they leave, and XTM One sums it up for the CISO. A security validation engineer drives it, with the SOC manager who asks the question in OpenCTI. The steps below are the pages to follow, in order.

::diagram{id=journey-threat-informed-validation}

## Who takes part

- **Security validation engineers** prepare OpenAEV: the agents on the endpoints, the collector of the EDR, the rule that aims the tests, and the payloads the scenario lacks (steps 1 to 3, 5 and 6).
- **SOC managers** ask for the security coverage of the threat in OpenCTI, then read the scores there and in OpenAEV (steps 4, 6 and 7).
- **CTI analysts** keep the threat current in OpenCTI: the techniques they link to it are what OpenAEV tests.
- The risk manager reads the price of the gaps in OpenCRQ (step 8), and the CISO receives the summary XTM One writes (step 9).

## Why it spans several modules

Each module answers one part of the question "would we stop it today?":

- **OpenCTI** knows the threat: the intrusion set, the report, the attack patterns and vulnerabilities it uses. A security coverage turns that knowledge into a request to test, and holds the scores next to the intelligence that raised the question.
- **OpenAEV** tests it: one inject per technique, run by its agents on your own endpoints, on a schedule, and scored on prevention and detection by what your security tools saw.
- **OpenCRQ** prices it: each technique your tools did not prevent, or did not detect, becomes a finding, and where a control held less often than you declared, OpenCRQ lowers it on that technique.
- **XTM One** reads the three with your rights and writes the readout.

Alone, OpenCTI says what could happen, OpenAEV says what happened on one run, and OpenCRQ prices assumptions. Together, the intelligence chooses the tests, the tests correct the price, and the loop runs again when the threat or your defenses change. Why a simulation scores what it scores is in [How you validate your defenses in OpenAEV](/openaev/concepts/validation/); why a test result moves a price is in [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## What you need

- **OpenCTI**, with the threat to test linked to its attack patterns or vulnerabilities, and OpenAEV connected to it: the **OpenAEV Coverage** connector shows among the connectors of OpenCTI. See [Connect OpenAEV to OpenCTI to test security coverages](/openaev/use/coverage-enrichment/).
- **OpenAEV**, with OpenAEV agents on the endpoints to test, in asset groups, and a collector that reads your EDR or SIEM.
- **OpenCRQ**, for the price, with OpenAEV connected to the tenant by an administrator with **Manage Security**. See [Import OpenAEV results](/opencrq/use/sources/openaev/).
- **XTM One**, for the summary, with the platforms registered with it. **Ask Ariane** requires the Enterprise Edition. See [Connect the modules to each other](/deploy/configure/connect-modules/).
- Capabilities:
  - in OpenCTI, **Create / Update knowledge**, to create the coverage;
  - in OpenAEV, **Access assessment** to read the scenario, **Manage assessment** to change its injects, **Manage tenant settings** for the default asset rules, and **Manage threat arsenal** to add a payload;
  - in OpenCRQ, any member of the tenant reads the findings and the risk scenarios.

## How it unfolds

The story: on the night of 9 January 2026, Velvet Harbor reached Grand Meridian Hotels through a fake booking request for the European Hospitality Summit 2026. A PowerShell loader ran on a front-desk workstation, and guest records were copied from the booking database server. Example CERT published the report "Velvet Harbor targets hotel booking systems" in OpenCTI, with three techniques: Spearphishing Attachment (T1566.001), PowerShell (T1059.001) and Web Protocols (T1071.001). Hélène Marchetti, the CISO, asks one question: would Meridian EDR stop it today?

### Prepare the ground

OpenAEV agents run on the 40 front-desk workstations of the French and Spanish hotels, in the asset group `Front-desk workstations` ([step 1](/openaev/use/agents/)), and the CrowdStrike collector reads Meridian EDR, so that each test is scored by what the EDR saw ([step 2](/openaev/use/collectors/)). A coverage names platforms, such as Windows, not machines: by default, a Windows test runs on every Windows endpoint of the tenant. The validation engineer narrows the rule of the `security coverage: windows` tag to the front desks before anything is created, because an inject keeps the targets it was built with ([step 3](/openaev/configure/asset-rules/)).

### Ask the question in OpenCTI

The SOC manager opens the report and adds an automated security coverage, every week, on Windows endpoints ([step 4](/opencti/use/analyze/security-coverage/)). Within seconds, OpenAEV builds the scenario `Coverage of Velvet Harbor targets hotel booking systems`: an inject for PowerShell and one for Web Protocols, and a disabled placeholder for Spearphishing Attachment, because no payload of the threat arsenal opens an attachment on Windows x86_64. The validation engineer adds that payload and the SOC manager refreshes the coverage: the placeholder becomes a real inject ([step 5](/openaev/use/scenarios/from-opencti/)).

### Let the attacks run

The first simulation starts 2 minutes after the scenario is created, then every week ([step 6](/openaev/use/simulations/)). Each inject waits for its expectations: did Meridian EDR prevent it, did it detect it ([step 7](/openaev/use/expectations/))? The run reads **Prevention** 67 and **Detection** 100: Meridian EDR stops the lure and the loader, and sees the command and control traffic without blocking it. A minute or two after the simulation finishes, the same scores reach the coverage in OpenCTI, where **Attack patterns coverage** shows Web Protocols as the technique left to fix.

### Price the gap and tell the CISO

Every hour, OpenCRQ reads the finished simulations: the Web Protocols result becomes a finding on the front-desk workstations, and the risk scenario Data Breach on Guest Records is priced again, the controls lowered on the techniques they did not stop ([step 8](/opencrq/use/sources/openaev/)). On the coverage or on the simulation, **Ask Ariane** answers "Summarize for the CISO how our front desks stand against Velvet Harbor, what is still missing, and what it costs" ([step 9](/xtm-one/use/in-openaev/)).

When the SOC blocks the command and control traffic, the next weekly simulation prevents Web Protocols: OpenCRQ dismisses the finding as **Not exposed**, prevented by the EDR in that simulation, and the scenario is priced again. Nobody restarts the journey: it runs until the **Duration** of the coverage ends.

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Run a CTEM cycle across the platform](/workflows/ctem-cycle/), which uses these results to choose what to fund.
- [Keep your detections tested every week](/workflows/validate-detections/), to follow the detection score of your EDR over time.
- [Follow your priority threats with PIRs](/opencti/use/analyze/pir/), to choose which threats to cover first.
