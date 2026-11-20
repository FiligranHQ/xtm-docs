---
title: Run actions through Caldera
description: Connect OpenAEV to a MITRE Caldera server, install Caldera agents on your endpoints from OpenAEV, and run attack actions through them.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/inject-caldera/
---

The Caldera executor runs OpenAEV implants through the agents of a [MITRE Caldera](https://caldera.mitre.org/) server. OpenAEV gives you the command that installs a Caldera agent on each endpoint, imports the agents Caldera knows as endpoints, and, for each inject, asks Caldera to run an ability on the agent that starts the implant. Caldera serves only as a neutral way to reach the endpoints: the attack actions come from the Threat Arsenal of OpenAEV, like with any other executor. This page deploys Caldera next to OpenAEV, connects them, and installs the agents. To compare it with the OpenAEV agent and the EDR executors, see [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=openaev-caldera-flow}

## Before you start

- A Caldera server that OpenAEV reaches, and that the endpoints reach too, at an address of its own if needed. The steps below deploy one.
- To deploy the executor in the interface: an Enterprise Edition license and the **Manage tenant settings** capability. Without Enterprise Edition, the executor can only come from the configuration of a new platform: see the example.
- To install the agents: the **Install agent** capability, and administrator rights on the endpoints.

## Deploy Caldera

Add a Caldera service to the `docker-compose.yml` of OpenAEV, as in the [docker-compose.caldera.yml](https://github.com/OpenAEV-Platform/docker/blob/master/docker-compose.caldera.yml) of the OpenAEV Docker repository:

```yaml
services:
  caldera:
    image: openaev/caldera-server:5.1.0
    restart: always
    ports:
      - "8888:8888"
    environment:
      CALDERA_URL: http://localhost:8888
    volumes:
      - type: bind
        source: caldera.yml
        target: /usr/src/app/conf/local.yml
```

Caldera reads its settings from a file. Download [caldera.yml](https://github.com/OpenAEV-Platform/docker/blob/master/caldera.yml) next to `docker-compose.yml` and change the values its comments mark **Change this**: the passwords of the `red` and `blue` users, the API keys `api_key_red`, `api_key_blue` and `api_key`, `crypt_salt`, `encryption_key`, the SSH tunnel password, and `app.contact.http`, the address where the agents reach Caldera. Then run `docker compose up -d`.

## Connect OpenAEV to Caldera

:::steps
1. Open :menu[Integrations], then the **Available** tab.
2. Open **Caldera Executor** and click **Deploy**. The **Create a new connector instance** panel opens.
3. Fill in the form. Each field is named after its configuration key:

   | Field | What to enter |
   | --- | --- |
   | **Display name** | The executor's name in OpenAEV, `Caldera` by default |
   | **Executor Caldera Url** | The address where OpenAEV reaches Caldera, such as `http://caldera:8888` on the Docker network |
   | **Executor Caldera Public Url** | The address where the endpoints reach Caldera: the same as `app.contact.http` |
   | **Executor Caldera Api Key** | The red API key of `caldera.yml`, `api_key_red` |

4. Click **Create**.
5. In the **Deployed** tab, open the executor and click **Start**.
:::

When started, the executor creates in Caldera the abilities that start the implant of each injector, then reads the agents of Caldera every 60 seconds. When Caldera does not answer, OpenAEV shows the banner **Executor Caldera is not responding, your exercises may be impacted.**

::screenshot{id=openaev-executor-caldera-form alt="The Create a new connector instance panel of the Caldera executor: its display name and instance name, the settings it needs, each with its help."}

## Install the Caldera agent on an endpoint

:::steps
1. In the top bar, click the **Install simulation agents** icon, then **Install Caldera**.
2. On **Choose your platform**, click **Install Windows agent**, **Install Linux agent** or **Install MacOS agent**. On macOS, choose the processor, **x86_64** or **arm64**.
3. Under **Step 1 - Install the agent**, click **Copy** or **Download**, then run the command on the endpoint: in a PowerShell prompt run as administrator on Windows, as root on Linux and macOS.
4. Under **Step 2 - Add antivirus exclusions**, copy the folder, the file and the hashes shown, and add them as exclusions in your antivirus. Antivirus tools detect and block the Caldera agent, Sandcat, by default.
:::

The command downloads the agent from OpenAEV, in `C:\Program Files (x86)\Filigran\OAEV Caldera` on Windows and `/opt/openaev-caldera-agent` on Linux and macOS, and starts it in the Caldera group `red`. On macOS, the command creates `/opt/openaev` but writes the agent to `/opt/openaev-caldera-agent`: create that folder first, with `sudo mkdir -p /opt/openaev-caldera-agent`. On Windows, a scheduled task, `OpenAEVCaldera`, starts it again at each logon. On Linux and macOS, it does not start again after a reboot: add it to `rc.local` or an equivalent to keep it running.

## Check that it works

Within a minute, open :menu[Assets]: the endpoint shows in the **Endpoints** list with **Caldera** in its **Executors** column. In Caldera, the agent shows in the `red` group.

To uninstall the agent on Windows, run in a PowerShell prompt as administrator:

```powershell
schtasks /delete /tn OpenAEVCaldera
Stop-Process -Name oaev-agent-caldera
rm -force -Recurse "C:\Program Files (x86)\Filigran\OAEV Caldera"
```

:::example[A Caldera lab for Grand Meridian's purple team]
Grand Meridian's purple team already runs Caldera at `https://caldera.grand-meridian.example` for its own operations, and the platform runs OpenAEV without an Enterprise Edition license. The platform engineer adds the Caldera executor to the configuration of a new OpenAEV platform before its first start, which turns it into a deployed executor:

```bash
EXECUTOR_CALDERA_ENABLE=true
EXECUTOR_CALDERA_URL=http://caldera:8888
EXECUTOR_CALDERA_PUBLIC-URL=https://caldera.grand-meridian.example
EXECUTOR_CALDERA_API-KEY=${CALDERA_API_KEY_RED}
```

On a running platform with Enterprise Edition, the same values go in the form of **Caldera Executor**. The validation engineer then installs the Caldera agent on a front-desk workstation of the Paris hotel, `fd-paris-02`, adds the exclusions of **Step 2 - Add antivirus exclusions** to Meridian EDR, and replays Velvet Harbor's PowerShell actions on it. The keys are in the [`executor.caldera` block](/reference/openaev/configuration/#executor-caldera).
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): the other executors, implant cleanup and troubleshooting.
- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/): the native agent, which runs as a service on every system.
- [Threat Arsenal](/openaev/use/threat-arsenal/): the actions an executor runs.
