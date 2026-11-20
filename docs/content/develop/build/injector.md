---
title: Build an OpenAEV injector
description: Write an external OpenAEV injector in Python, from its contract to the result of each inject, run it against your platform, see its inject type in an atomic test, and package it with Docker.
doc_type: how-to
products:
  - openaev
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-09
migrated_from:
  - openaev:/development/injectors/
---

This page builds an external injector that adds a new inject type to OpenAEV: **Send a fake booking request**, which posts a booking request carrying a suspicious link to the booking API of Grand Meridian Hotels, the way Velvet Harbor lures front-desk staff. It talks to the platform directly, with the REST API and RabbitMQ, in under 200 lines of Python, so you see every exchange and can port it to another language. Read [Injectors](/develop/integration-model/injectors/) first for the protocol it follows.

::diagram{id=develop-build-injector}

## Before you start

- An OpenAEV 3.261029.0 platform, here at `https://openaev.example.com`, and the identifier of the tenant the injector serves. The address of the interface shows it after the host; a platform with one tenant has the default tenant, `2cffad3a-0001-4078-b0e2-ef74274022c3`.
- A user for the injector, with a role that grants **Manage tenant settings** (to register) and **Manage assessment** (to report results), and its API token from its profile.
- RabbitMQ reachable from where the injector runs, under the host name and port of the platform's `openaev.rabbitmq.hostname` and `openaev.rabbitmq.port` ([`openaev.rabbitmq` block](/reference/openaev/configuration/#openaev-rabbitmq)): the platform hands these to the injector.
- Python 3.10 or later, and Docker for the last step.

## Lay out the project

```text
booking-probe/
├── src/
│   ├── contract.py
│   ├── main.py
│   └── requirements.txt
└── Dockerfile
```

`src/requirements.txt`:

```text
requests~=2.33
pika~=1.3
```

## Describe the contract

The contract is what the user sees: the inject type in the list, and the form with its fields. Its `contract_id` must never change once scenarios use it: a contract that disappears from the registration is deleted with its injects. The MITRE ATT&CK identifier links the contract to the technique, when the tenant has the MITRE ATT&CK knowledge base.

`src/contract.py`:

```python
import json

INJECTOR_TYPE = "grand_meridian_booking_probe"
CONTRACT_ID = "5d0c3a8e-2f4b-4c39-9a51-7f2e8c1b6d40"


def contract_content() -> dict:
    """The contract as the interface reads it: the form of the inject."""
    return {
        "contract_id": CONTRACT_ID,
        "config": {
            "type": INJECTOR_TYPE,
            "label": {"en": "Booking probe", "fr": "Sonde de réservation"},
            "color_dark": "#7e57c2",
            "color_light": "#5e35b1",
        },
        "label": {
            "en": "Send a fake booking request",
            "fr": "Envoyer une fausse demande de réservation",
        },
        "manual": False,
        "fields": [
            {
                "key": "booking_url",
                "label": "Booking API URL",
                "type": "text",
                "mandatory": True,
                "readOnly": False,
                "defaultValue": "https://booking.example.com/api/requests",
            },
            {
                "key": "attachment_url",
                "label": "Link in the request",
                "type": "text",
                "mandatory": True,
                "readOnly": False,
                "defaultValue": "https://reservations-confirm.example.com/booking-4821.pdf",
            },
        ],
        "variables": [],
        "context": {},
    }


def registration_contracts() -> list[dict]:
    """The contracts as the registration sends them."""
    content = contract_content()
    return [
        {
            "contract_id": CONTRACT_ID,
            "contract_labels": content["label"],
            "contract_attack_patterns_external_ids": ["T1566.002"],
            "contract_content": json.dumps(content),
            "contract_platforms": ["Service"],
            "is_atomic_testing": True,
        }
    ]
```

## Register, then keep registering

The injector registers at start, and again every 40 seconds: the interface shows it as stopped when its last registration is more than two minutes old. The answer carries the RabbitMQ credentials of the platform; never log it.

`src/main.py`, first part:

```python
import json
import os
import ssl
import threading
import time

import pika
import requests

from contract import INJECTOR_TYPE, registration_contracts

OPENAEV_URL = os.environ["OPENAEV_URL"].rstrip("/")
OPENAEV_TOKEN = os.environ["OPENAEV_TOKEN"]
OPENAEV_TENANT_ID = os.environ["OPENAEV_TENANT_ID"]
INJECTOR_ID = os.environ["INJECTOR_ID"]
INJECTOR_NAME = os.environ.get("INJECTOR_NAME", "Booking probe")

API = f"{OPENAEV_URL}/api/tenants/{OPENAEV_TENANT_ID}"
HEADERS = {"Authorization": f"Bearer {OPENAEV_TOKEN}"}
PING_SECONDS = 40


def register() -> dict:
    registration = {
        "injector_id": INJECTOR_ID,
        "injector_name": INJECTOR_NAME,
        "injector_type": INJECTOR_TYPE,
        "injector_contracts": registration_contracts(),
        "injector_author": "Grand Meridian Hotels",
    }
    response = requests.post(
        f"{API}/injectors",
        headers=HEADERS,
        files={"input": (None, json.dumps(registration), "application/json")},
        timeout=30,
    )
    response.raise_for_status()
    return response.json()


def keep_registering() -> None:
    while True:
        time.sleep(PING_SECONDS)
        try:
            register()
        except requests.RequestException as error:
            print(f"Registration failed, next try in {PING_SECONDS} s: {error}")
```

## Run each inject and report it

Each message of the queue is one inject. `inject_content` holds the values the user entered, by the `key` of each field. The injector tells the platform it received the inject, which moves it to **Pending**, does the work, then posts the result with `complete`, which closes it. It acknowledges the message last, so an inject is not lost if the injector stops halfway.

`src/main.py`, second part:

```python
def report(inject_id: str, step: str, body: dict) -> None:
    response = requests.post(
        f"{API}/injects/execution/{step}/{inject_id}", headers=HEADERS, json=body, timeout=30
    )
    response.raise_for_status()


def handle(channel, method, properties, body) -> None:
    inject = json.loads(body)["injection"]
    inject_id = inject["inject_id"]
    values = inject.get("inject_content") or {}
    report(inject_id, "reception", {"tracking_total_count": 1})

    started = time.monotonic()
    try:
        answer = requests.post(
            values["booking_url"],
            json={
                "guest_name": "Laurent Diaz",
                "arrival": "2026-01-23",
                "nights": 2,
                "message": "Please confirm my booking with the attached document.",
                "attachment": values["attachment_url"],
            },
            timeout=20,
        )
        status, message = "SUCCESS", f"The booking API answered HTTP {answer.status_code}."
    except requests.RequestException as error:
        status, message = "ERROR", f"The booking API could not be reached: {error}"

    report(
        inject_id,
        "callback",
        {
            "execution_message": message,
            "execution_status": status,
            "execution_duration": int((time.monotonic() - started) * 1000),
            "execution_action": "complete",
        },
    )
    channel.basic_ack(delivery_tag=method.delivery_tag)


def connect(info: dict) -> pika.BlockingConnection:
    tls = pika.SSLOptions(ssl.create_default_context(), info["host"]) if info["use_ssl"] else None
    return pika.BlockingConnection(
        pika.ConnectionParameters(
            host=info["host"],
            port=info["port"],
            virtual_host=info["vhost"],
            credentials=pika.PlainCredentials(info["user"], info["pass"]),
            ssl_options=tls,
        )
    )


if __name__ == "__main__":
    registration = register()
    threading.Thread(target=keep_registering, daemon=True).start()
    channel = connect(registration["connection"]).channel()
    channel.basic_qos(prefetch_count=1)
    channel.basic_consume(queue=registration["listen"], on_message_callback=handle)
    print(f"{INJECTOR_NAME} waits for injects on {registration['listen']}")
    channel.start_consuming()
```

## Run it

:::steps
1. Generate the injector's identifier once, and keep it: `python3 -c 'import uuid; print(uuid.uuid4())'`.
2. Set the variables and start the injector, as in the example below. It prints the queue it listens to.
3. In OpenAEV, open :menu[Integrations], **Deployed** tab. The injector shows under its name, **Started**, with the time the platform last heard from it.

   ::screenshot{id=openaev-integrations-injectors-deployed alt="The Deployed tab of Integrations filtered on injectors lists each injector with its status, the built-in ones marked Built-in; a registered external injector joins the list under its name."}

4. Open :menu[Atomic testings] and click **Create**. Search for **Send a fake booking request**, choose it, check the two fields and launch the test.
:::

:::example[Probe the booking API with a Velvet Harbor lure]
The red team of Grand Meridian Hotels wants to know whether the web application firewall in front of the booking API stops a booking request that carries a link to a look-alike confirmation page, as Velvet Harbor sent to the Paris front desk in January 2026. A developer runs the injector next to the staging booking API:

```bash
cd booking-probe/src
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
export OPENAEV_URL=https://openaev.example.com
export OPENAEV_TOKEN="$BOOKING_PROBE_OPENAEV_TOKEN"
export OPENAEV_TENANT_ID=2cffad3a-0001-4078-b0e2-ef74274022c3
export INJECTOR_ID=9f2d6c1e-4b7a-4e0c-8d5f-3a1b2c4d5e6f
export INJECTOR_NAME="Booking probe"
python3 main.py
```

The atomic test **Send a fake booking request** keeps the default URL `https://booking.example.com/api/requests` and the link `https://reservations-confirm.example.com/booking-4821.pdf`. Its **Execution details** tab reads `The booking API answered HTTP 403.`: the firewall blocked the request. An answer `HTTP 201` would mean the lure reached the booking system.
:::

## Check that it works

- The atomic test ends **Executed** when the request was sent, or **Error** when the booking API could not be reached, and its **Execution details** tab shows the message of the callback.
- An inject that stays **Pending** for more than 10 minutes, `inject.execution.threshold.minutes`, times out: the injector received it but never posted `complete`. Look at the injector's output for an error of the callback.
- The registration fails with HTTP 404: the user's role lacks **Manage tenant settings**. OpenAEV answers a missing right with 404, not 403. A 403 `TENANT_ACCESS_DENIED` means the user is not a member of the tenant in the path.
- An inject that stays **On-going** without moving to **Pending** never reached the injector: check that the injector is **Started** and that it reaches RabbitMQ under the host name the registration returned.

## Package it

`Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /opt/booking-probe
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
CMD ["python", "main.py"]
```

```bash
docker build -t grand-meridian/injector-booking-probe:1.0.0 .
docker run -d --name booking-probe \
  -e OPENAEV_URL -e OPENAEV_TOKEN -e OPENAEV_TENANT_ID -e INJECTOR_ID -e INJECTOR_NAME \
  grand-meridian/injector-booking-probe:1.0.0
```

The variables are the names the Filigran injectors use, so the same image can later be described in a catalog and deployed by [XTM Composer](/deploy/integrations/composer/). [Run an integration without XTM Composer](/deploy/integrations/manual/) covers running it next to the platform.

## Go further

- Let the user choose expectations: add a field of type `expectation`, and collectors answer them. See [Collectors](/develop/integration-model/collectors/).
- Report targets: when an inject targets assets, put their identifiers in `execution_context_identifiers`, so each trace shows on its target.
- Return findings: post a JSON document in `execution_output_structured` for the contract's outputs.
- Use `pyoaev`, Filigran's Python library, which wraps the registration, the heartbeat and the queue for the injectors of [OpenAEV-Platform/injectors](https://github.com/OpenAEV-Platform/injectors).

## What's next

- [Injectors](/develop/integration-model/injectors/), every field of the protocol.
- [Build an OpenAEV collector](/develop/build/collector/), to read what the security tools saw of the inject.
