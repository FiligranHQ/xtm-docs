---
title: Install the OpenAEV agent on your endpoints
description: Install the OpenAEV agent on Windows, Linux and macOS endpoints, choose how it runs, add its antivirus exclusion, and check that the endpoint enrolls.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/openaev-agent/
---

The OpenAEV agent is the program you install on an endpoint so that simulations can run attack actions on it. It enrolls the endpoint as an asset of OpenAEV, fetches the jobs of each inject, and starts an implant that runs the actions: the agent itself does nothing on the endpoint, so your antivirus and endpoint detection and response (EDR) tools judge the actions, not the agent. This page installs it on Windows, Linux and macOS, from the command OpenAEV gives you. Where the agent sits among executors, injectors and collectors is in [How OpenAEV works](/openaev/concepts/architecture/#agents-and-executors).

::diagram{id=openaev-agents-install}

## Before you start

- Your role has the **Install agent** capability. Without it, the **Install simulation agents** icon does not show in the top bar.
- The endpoint runs on an x86_64 or ARM64 processor, with the service manager of its system: the Service Control Manager on Windows, systemd on Linux, launchd on macOS. 32-bit systems are not supported.
- The endpoint supports TLS 1.2 or later. On Linux and macOS, `curl` and `openssl` are installed: the installer stops if `openssl` is missing, because it checks the signature of the agent before installing it.
- The endpoint reaches the platform over HTTPS, outbound only. The agent uses `openaev.agent-url`, or `openaev.base-url` when the first is not set. On Windows, the install command also downloads its installer from `openaev.base-url`. Both keys are in the [`openaev` block](/reference/openaev/configuration/#openaev) of the configuration.
- For an advanced installation, you hold administrator rights on the endpoint: Administrator on Windows, root or `sudo` on Linux and macOS.

| System | Supported | Best effort |
| --- | --- | --- |
| Windows | Windows 10 and 11, Windows Server 2019 and 2022 | Windows 8 and 8.1, Windows Server 2016 and 2012 R2 |
| Linux | Distributions that run systemd, such as Debian and Ubuntu | |
| macOS | Apple silicon (ARM64) | Intel (x86_64) |

Best effort means the agent may work but the system is not validated: fixes for it are not guaranteed. Windows 7, Windows Server 2008 and Linux distributions without systemd are not supported.

::screenshot{id=openaev-agents-executors alt="The Agents page: the Install OpenAEV Agent card, for Windows, Linux and macOS."}

## Choose how the agent runs

The installation mode decides when the agent runs and with which rights the actions run. A simulation measures what an attacker could do with those rights, so pick the mode that matches the attacker you simulate.

| Mode | On screen | Runs | Actions run with |
| --- | --- | --- | --- |
| Session | **Standard Installation** tab | In the background, while the user who installed it is logged in | The rights of that user. On Windows, administrator rights when you install from an elevated PowerShell prompt |
| User service | **Advanced Installation** tab, **Install Agent as User** | As a service, from the moment the endpoint starts | The rights of the user you name in the command |
| System service | **Advanced Installation** tab, **Install Agent as System** | As a service, from the moment the endpoint starts | SYSTEM on Windows, root on Linux and macOS |

On macOS, only the system service is available.

You can install several agents on one endpoint, one per mode and user, for example a session agent of a standard user next to a system service. Each one shows on the endpoint's page in OpenAEV, and an inject runs on each of them, so you compare what each level of rights lets an attacker do.

## Install the agent

:::steps
1. In the top bar, click the **Install simulation agents** icon. The **Agents** page shows a card for each executor you can install from.
2. Click **Install OpenAEV Agent**.
3. On **Choose your platform**, click **Install Windows agent**, **Install Linux agent** or **Install MacOS agent**.
4. On **Installation Instructions**, choose the **Standard Installation** or the **Advanced Installation** tab. In **Advanced Installation**, choose **Install Agent as User** or **Install Agent as System**.
5. Click **Copy** to copy the command, or **Download** to save it as `openaev.ps1` or `openaev.sh`. The command carries the installer token of your tenant: treat it as a secret.
6. For **Install Agent as User**, replace the placeholders of the command with the account the agent runs as: `-User USER -Password PASSWORD` on Windows, with the domain in the user name, and `--user USER --group GROUP` on Linux.
7. Run the command on the endpoint:
   - Windows, **Standard Installation**: in a PowerShell prompt of the user. An elevated prompt gives the agent administrator rights.
   - Windows, **Advanced Installation**: in a PowerShell prompt run as administrator. For **Install Agent as User**, the account needs the **Log on as a service** right first.
   - Linux and macOS, **Standard Installation**: in a terminal of the user. The command ends with `| sh`.
   - Linux and macOS, **Advanced Installation**: in a terminal of an account that can use `sudo`. The command ends with `| sudo sh`.
:::

The installer checks the signature of the agent, refuses an older version than the one installed, installs the agent in its folder and starts it. The agent then registers with OpenAEV.

| System | Mode | Folder | Service or process |
| --- | --- | --- | --- |
| Windows | Session | `$HOME\.openaev\OAEVAgent-Session-<user>`, or `OAEVAgent-Session-Administrator-<user>` from an elevated prompt | Process `openaev-agent.exe`, started at logon |
| Windows | User service | `$HOME\.openaev\OAEVAgent-Service-<user>` | Service `OAEVAgent-Service-<user>` |
| Windows | System service | `C:\Program Files (x86)\Filigran\OAEV Agent` | Service `OAEVAgentService` |
| Linux | Session | `$HOME/.local/openaev-agent-session` | User unit `openaev-agent-session` |
| Linux | User service | `~<user>/.local/openaev-agent-service-<user>` | Unit `<user>-openaev-agent` |
| Linux | System service | `/opt/openaev-agent` | Unit `openaev-agent` |
| macOS | System service | `/opt/openaev-agent` | Launch daemon `io.filigran.openaev-agent` |

`<user>` is the user name without characters such as `\` and `/`.

::screenshot{id=openaev-agents-install-linux-session alt="The Standard Installation tab of the OpenAEV Agent for Linux: the agent runs as a session of the user, and the command fetches the installer with the installer token and pipes it to sh, with Copy and Download."}

## Check that the endpoint enrolled

Open :menu[Assets]. The **Endpoints** list shows the endpoint with **OpenAEV Agent** in its **Executors** column. Open the endpoint: each of its agents shows its **Privilege** (**Admin** or **Standard**), its **Deployment** (**Session** or **Service**), its **Version** and when it was **Last Seen**.

An agent that has not reached the platform for one hour shows as inactive, and injects skip it. If the endpoint does not show, see [Troubleshoot an agent](#troubleshoot-an-agent).

::screenshot{id=openaev-endpoint-agents alt="The page of fd-paris-01 with its OpenAEV agent: running as the Windows system account, Admin privilege, Service deployment, version 2.4.0, seen moments ago and Active."}

## Add the antivirus exclusion

Each inject runs in its own folder under `runtimes`, in the folder of the agent. Exclude this `runtimes` folder, and only this one, from your antivirus and EDR scans, for example `C:\Program Files (x86)\Filigran\OAEV Agent\runtimes` for a system service on Windows. The implant then starts and reports its results, while the files and commands of the attack actions stay visible to your tools: they are written elsewhere, under `payloads`, so detecting and blocking them is what the simulation measures.

On Windows, the command that starts an implant also creates two Windows Defender Firewall rules for the implant's executable, `Allow OpenAEV Inbound` and `Allow OpenAEV Outbound`, replacing those of the previous implant. If a group policy manages the firewall rules of your endpoints, expect these two names in it.

## Install behind a proxy

Set `HTTP_PROXY` and `HTTPS_PROXY` on the endpoint before you run the command: as system variables, or in the same terminal session. For a service, the account of the service needs them too. The agent uses them only when `openaev.with-proxy` is `true` on the platform at the moment you copy the command.

When the endpoint does not trust the platform's certificate, such as a certificate of an internal authority, set `openaev.unsecured-certificate` to `true` on the platform, then copy the command again. Both keys are in the [`openaev` block](/reference/openaev/configuration/#openaev).

## Keep agents running well

| Setting | Default | Effect |
| --- | --- | --- |
| `executor.openaev.agent.max-simultaneous-jobs` | `5` | Jobs an agent runs at the same time, so a simulation does not overload the endpoint |
| `executor.openaev.agent.auto-update-enabled` | `true` | When `false`, the platform stops sending upgrade jobs to its agents. It takes effect with an active Enterprise Edition license only |
| `openaev.agent.queue-threshold` | `0` | Pending jobs above which an agent counts as overloaded. `0` turns the check off |

The first two are in the [`executor.openaev` block](/reference/openaev/configuration/#executor-openaev), the third in the [`openaev.agent` block](/reference/openaev/configuration/#openaev-agent).

The agent also cleans its folders on its own: it stops the implants that run too long, then deletes their folders. Its thresholds are in the `[cleanup]` section of `openaev-agent-config.toml`, in the agent's folder.

## Protect the installer token

The command embeds the installer token of your tenant. It belongs to the tenant's service account, and lets the agent and its implants fetch their jobs, download the files of an action by their exact ID, send results and download the agent installer. It gives access to nothing else.

The **Install agent** capability lets a user read the token, not change it. If the token leaks, a user with **Manage tenant users, groups and roles** rotates it through the API, with `POST /api/agent/installer/openaev/token/rotate`. The rotation cuts off every agent already installed in the tenant: install them again with a new command.

## Uninstall the agent

Uninstalling the agent does not remove the endpoint from OpenAEV. Uninstall the agent first, since a running agent registers again and creates the endpoint again.

::::tabs
:::tab[Windows]
1. For a session installation, stop the agent:

   ```powershell
   Get-Process openaev-agent | Where-Object { $_.Path -eq "$HOME\.openaev\OAEVAgent-Session-<user>\openaev-agent.exe" } | Stop-Process -Force
   ```

2. Run `uninstall.exe` from the agent's folder and confirm. It removes the service or the startup task, then the folder. For a service, run it as administrator.
3. For a user service, withdraw the **Log on as a service** right from the account if it no longer needs it.
:::
:::tab[Linux]
Session:

```bash
systemctl --user stop openaev-agent-session
systemctl --user disable openaev-agent-session
systemctl --user daemon-reload
systemctl --user reset-failed
rm -rf $HOME/.local/openaev-agent-session
```

User service, with the user name in place of `<user>`:

```bash
sudo systemctl stop <user>-openaev-agent
sudo systemctl disable <user>-openaev-agent
sudo systemctl daemon-reload
sudo systemctl reset-failed
sudo rm -rf ~<user>/.local/openaev-agent-service-<user>
```

System service:

```bash
sudo systemctl stop openaev-agent
sudo systemctl disable openaev-agent
sudo systemctl daemon-reload
sudo systemctl reset-failed
sudo rm -rf /opt/openaev-agent
```
:::
:::tab[macOS]
```bash
sudo launchctl bootout system /Library/LaunchDaemons/io.filigran.openaev-agent.plist
sudo rm -f /Library/LaunchDaemons/io.filigran.openaev-agent.plist
sudo rm -rf /opt/openaev-agent
```
:::
::::

Then remove the endpoint: in :menu[Assets], open the menu of the endpoint and click **Delete**. It takes the **Delete assets** capability.

## Troubleshoot an agent

The agent writes its log to `openaev-agent.log`, in its folder. Each implant gets its own folder under `runtimes`, with its executable and its own logs.

When the endpoint does not show in :menu[Assets]:

1. **The agent runs.** Check the service or the process of the table above, for example `systemctl status openaev-agent` or `Get-Service OAEVAgentService`.
2. **The log shows no registration error.** Look for `Fail registering the agent` in `openaev-agent.log`, followed by its cause.
3. **The endpoint reaches the URL.** The `url` value of `openaev-agent-config.toml`, in the agent's folder, comes from `openaev.agent-url` or `openaev.base-url`. If it is wrong, correct the platform's configuration, then install again with a new command.
4. **You look in the right tenant.** The command carries the tenant you were in when you copied it: the endpoint shows in that tenant only.
5. **The certificate and the proxy are accepted.** See [Install behind a proxy](#install-behind-a-proxy).

:::example[Enroll the front-desk workstations of Grand Meridian]
Grand Meridian Hotels wants to replay Velvet Harbor's loader on the front-desk workstations of its Paris hotel, the Windows 11 machines where the night auditor Paul Garnier works. The attacker ran with the rights of the workstation, so the validation engineer installs the agent as a system service.

The platform engineer first makes sure the workstations reach OpenAEV through the hotel's proxy:

```bash
OPENAEV_BASE-URL=https://openaev.example.com
OPENAEV_WITH-PROXY=true
```

Then, for each workstation:

| Step | Value |
| --- | --- |
| Executor | **Install OpenAEV Agent** |
| Platform | **Install Windows agent** |
| Tab | **Advanced Installation**, **Install Agent as System** |
| Run in | An administrator PowerShell prompt on `fd-paris-01.grand-meridian.example`, with `HTTPS_PROXY=http://proxy.grand-meridian.example:3128` set as a system variable |
| Exclusion in Meridian EDR | `C:\Program Files (x86)\Filigran\OAEV Agent\runtimes` |

A minute later, `fd-paris-01` shows in :menu[Assets] with **OpenAEV Agent**, **Admin** and **Service**. The simulation can now target the front-desk workstations, and Meridian EDR sees every PowerShell action of the loader.
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): run actions through an agent you already have.
- [Assets](/openaev/use/targets/assets/): group the endpoints your simulations target.
- [How collectors measure what your security tools saw](/openaev/use/collectors/): read back what your EDR detected.
- [OpenAEV configuration reference](/reference/openaev/configuration/).
