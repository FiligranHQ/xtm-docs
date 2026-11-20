---
title: Build an OpenAEV collector
description: Write an OpenAEV collector in Python that matches the detection and prevention expectations of your simulations with the alerts of your security tool, run it, and read its verdicts.
doc_type: how-to
products:
  - openaev
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-09
migrated_from:
  - openaev:/development/collectors/
---

This page builds a collector for a security tool OpenAEV has no collector for: the alert store of the Grand Meridian Hotels security operations center (SOC), **Meridian SOC**, which gathers the alerts of the endpoints with their process tree. At each run, the collector reads the detection and prevention expectations nobody answered for it, looks for the alerts of each one, and marks it **Detected** or **Prevented**, with the alerts as evidence. It calls the REST API of the platform only, in about 140 lines of Python. Read [Collectors](/develop/integration-model/collectors/) first for the protocol it follows.

::diagram{id=develop-build-collector}

## Before you start

- An OpenAEV 3.261029.0 platform, here at `https://openaev.example.com`, and the identifier of the tenant the collector serves: the address of the interface shows it after the host, and a platform with one tenant has the default tenant, `2cffad3a-0001-4078-b0e2-ef74274022c3`.
- A user for the collector, with a role that grants **Manage tenant settings**, **Manage security platforms** and **Manage assessment**, and its API token from its profile.
- Injects that run through an implant, with the OpenAEV agent or an executor of your EDR, and that carry **Detection** or **Prevention** expectations: the collector matches the name of the implant's process.
- The API of your tool. Here, `https://alerts.example.com/api/alerts?parent_process=<name>` returns the alerts whose process tree holds that name, each with `id`, `title`, `action` (`blocked` or `detected`), `created_at` and `url`.
- Python 3.10 or later.

## Lay out the project

```text
meridian-soc-collector/
├── src/
│   ├── main.py
│   └── requirements.txt
└── Dockerfile
```

`src/requirements.txt`:

```text
requests~=2.33
```

## Register with a security platform

The verdicts carry the name of a security platform. The collector creates its own once, keyed on the collector's identifier, then registers with it at every run: each registration is also its heartbeat, and the interface shows it as stopped after two minutes without one.

`src/main.py`, first part:

```python
import json
import os
import time
from datetime import datetime, timezone

import requests

OPENAEV_URL = os.environ["OPENAEV_URL"].rstrip("/")
OPENAEV_TOKEN = os.environ["OPENAEV_TOKEN"]
OPENAEV_TENANT_ID = os.environ["OPENAEV_TENANT_ID"]
COLLECTOR_ID = os.environ["COLLECTOR_ID"]
COLLECTOR_NAME = os.environ.get("COLLECTOR_NAME", "Meridian SOC")
ALERTS_API_URL = os.environ["ALERTS_API_URL"].rstrip("/")
ALERTS_API_TOKEN = os.environ["ALERTS_API_TOKEN"]

API = f"{OPENAEV_URL}/api/tenants/{OPENAEV_TENANT_ID}"
HEADERS = {"Authorization": f"Bearer {OPENAEV_TOKEN}"}
PERIOD_SECONDS = 60
LOOKBACK_MINUTES = 360  # the default expiration of detection and prevention: 6 hours


def upsert_security_platform() -> str:
    response = requests.post(
        f"{API}/security_platforms/upsert",
        headers=HEADERS,
        json={
            "asset_name": "Meridian SOC",
            "asset_description": "The alert store of the Grand Meridian Hotels SOC.",
            "asset_external_reference": COLLECTOR_ID,
            "security_platform_type": "SIEM",
        },
        timeout=30,
    )
    response.raise_for_status()
    return response.json()["asset_id"]


def register(security_platform_id: str) -> None:
    registration = {
        "collector_id": COLLECTOR_ID,
        "collector_name": COLLECTOR_NAME,
        "collector_type": "grand_meridian_soc",
        "collector_period": PERIOD_SECONDS,
        "collector_security_platform": security_platform_id,
    }
    response = requests.post(
        f"{API}/collectors",
        headers=HEADERS,
        files={"input": (None, json.dumps(registration), "application/json")},
        timeout=30,
    )
    response.raise_for_status()
```

## Match the expectations with the alerts

Each pending expectation carries signatures. `parent_process_name` is the name of the implant's process for this inject and this agent, `oaev-implant-<inject>-agent-<agent>`: an alert whose process tree holds it belongs to this inject on this endpoint. A detection expectation is met by any such alert; a prevention expectation by an alert whose action was blocked, and missed when the alerts show the action ran unblocked. An expectation without alerts stays pending: the platform closes it as **Not Detected** or **Not Prevented** when it expires.

`src/main.py`, second part:

```python
def pending_expectations() -> list[dict]:
    response = requests.get(
        f"{API}/injects/expectations/assets/{COLLECTOR_ID}",
        headers=HEADERS,
        params={"expiration_time": LOOKBACK_MINUTES},
        timeout=60,
    )
    response.raise_for_status()
    return response.json()


def implant_process(expectation: dict) -> str | None:
    for signature in expectation["inject_expectation_signatures"]:
        if signature["type"] == "parent_process_name":
            return signature["value"]
    return None


def alerts_for(process_name: str) -> list[dict]:
    response = requests.get(
        f"{ALERTS_API_URL}/alerts",
        headers={"Authorization": f"Bearer {ALERTS_API_TOKEN}"},
        params={"parent_process": process_name},
        timeout=30,
    )
    response.raise_for_status()
    return response.json()["alerts"]


def run_once() -> None:
    verdicts, traces = {}, []
    for expectation in pending_expectations():
        process_name = implant_process(expectation)
        alerts = alerts_for(process_name) if process_name else []
        if not alerts:
            continue
        expectation_id = expectation["inject_expectation_id"]
        if expectation["inject_expectation_type"] == "PREVENTION":
            evidence = [alert for alert in alerts if alert["action"] == "blocked"]
            success = bool(evidence)
            evidence = evidence or alerts
        else:
            evidence, success = alerts, True
        verdicts[expectation_id] = {
            "collector_id": COLLECTOR_ID,
            "is_success": success,
            "metadata": {"alert_id": evidence[0]["id"]},
        }
        traces += [
            {
                "inject_expectation_trace_expectation": expectation_id,
                "inject_expectation_trace_source_id": COLLECTOR_ID,
                "inject_expectation_trace_alert_name": alert["title"],
                "inject_expectation_trace_alert_link": alert["url"],
                "inject_expectation_trace_date": alert["created_at"],
            }
            for alert in evidence
        ]

    if verdicts:
        requests.put(
            f"{API}/injects/expectations/bulk", headers=HEADERS, json={"inputs": verdicts}, timeout=60
        ).raise_for_status()
    if traces:
        requests.post(
            f"{API}/inject-expectations-traces/bulk",
            headers=HEADERS,
            json={"expectation_traces": traces},
            timeout=60,
        ).raise_for_status()
    requests.put(
        f"{API}/collectors/{COLLECTOR_ID}",
        headers=HEADERS,
        json={"collector_last_execution": datetime.now(timezone.utc).isoformat()},
        timeout=30,
    ).raise_for_status()
    print(f"{len(verdicts)} verdict(s), {len(traces)} alert(s)")


if __name__ == "__main__":
    security_platform_id = upsert_security_platform()
    while True:
        try:
            register(security_platform_id)
            run_once()
        except requests.RequestException as error:
            print(f"Run failed, next try in {PERIOD_SECONDS} s: {error}")
        time.sleep(PERIOD_SECONDS)
```

## Run it

:::steps
1. Generate the collector's identifier once, and keep it: `python3 -c 'import uuid; print(uuid.uuid4())'`.
2. Set the variables and start the collector, as in the example below. It prints the number of verdicts at each run.
3. In OpenAEV, open :menu[Integrations], **Deployed** tab. The collector shows under its name, **Started**, with the time of its last run.

   ::screenshot{id=openaev-integrations-collectors-deployed alt="The Deployed tab of Integrations filtered on collectors lists each collector with its status and the time the platform last heard from it, next to the built-in expectation managers."}

4. Open :menu[Security platforms]: **Meridian SOC** is there, of type SIEM.
:::

:::example[Score the front-desk simulation with Meridian SOC]
The SOC of Grand Meridian Hotels wants its alert store to answer the expectations of the Velvet Harbor simulation on the front-desk workstations. A developer runs the collector next to the alert store:

```bash
cd meridian-soc-collector/src
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
export OPENAEV_URL=https://openaev.example.com
export OPENAEV_TOKEN="$MERIDIAN_SOC_OPENAEV_TOKEN"
export OPENAEV_TENANT_ID=2cffad3a-0001-4078-b0e2-ef74274022c3
export COLLECTOR_ID=3c7e1a52-8d4f-4b19-a6e0-5f2b9c8d7e61
export COLLECTOR_NAME="Meridian SOC"
export ALERTS_API_URL=https://alerts.example.com/api
export ALERTS_API_TOKEN="$MERIDIAN_SOC_ALERTS_TOKEN"
python3 main.py
```

The PowerShell loader of the simulation (T1059.001) runs on `fd-paris-01` and `fd-paris-02`. The alert store holds an alert **Suspicious PowerShell from a booking attachment** on `fd-paris-01`, action `blocked`, and the same alert on `fd-paris-02`, action `detected`. At its next run, the collector writes four verdicts: `fd-paris-01` reads **Detected** and **Prevented**, `fd-paris-02` **Detected** and **Not Prevented**, each with the alert and its link to the alert store, under the security platform **Meridian SOC**.
:::

## Check that it works

- On the inject's result, the **Detection** and **Prevention** results of each endpoint name **Meridian SOC**, and the alerts open in the alert store from their link.
- A run that prints `0 verdict(s)` while injects ran: check that the injects ran through an implant, and that the alert store finds alerts by the process name the expectation carries.
- An HTTP 404 on a call that exists means the user's role lacks the capability it needs: OpenAEV answers a missing right with 404, not 403, so as not to reveal what exists. A 403 `TENANT_ACCESS_DENIED` means the user is not a member of the tenant in the path.

## Package it

`Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /opt/meridian-soc-collector
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
CMD ["python", "main.py"]
```

Pass the same variables to the container. The collector only opens outbound connections, to OpenAEV and to the alert store, so it can run next to the alert store, on another network than the platform. [Run an integration without XTM Composer](/deploy/integrations/manual/) covers running it next to the platform.

## Go further

- Answer the expectations of other injectors: their process name is `<injector type><inject>-agent-<agent>`.
- Match without a process tree: a SIEM can match the addresses of the endpoint (`source_ipv4_address`) and the time window (`start_date`, `end_date`) instead.
- Use `pyoaev`, Filigran's Python library, which wraps the registration and the run loop for the collectors of [OpenAEV-Platform/collectors](https://github.com/OpenAEV-Platform/collectors).

## What's next

- [Collectors](/develop/integration-model/collectors/), every field of the protocol.
- [Build an OpenAEV injector](/develop/build/injector/), to add the inject a collector then scores.
- [Expectations](/openaev/use/expectations/), how the verdicts show to the validation engineer.
