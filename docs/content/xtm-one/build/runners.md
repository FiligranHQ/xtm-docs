---
title: Let agents act on a computer with XTM One Desktop
description: Enroll a workstation or a server as a runner, decide what agents may do on it, and give an agent the machines it may reach, for screenshots, commands, files and a signed-in browser.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/resources/runners/
---

A runner lets an AI agent act on a real computer: take a genuine full-desktop screenshot as audit evidence, run a command, read and write files, or drive a browser you already signed in to. The runner comes inside **XTM One Desktop**, the native XTM One app for Windows, macOS and Linux, which also gives you your chats in a window of their own. On servers and virtual machines, the headless runner does the same work without a window. This page enrolls a machine, sets what agents may do on it, and gives an agent the machines it may reach.

::diagram{id=xtm-one-build-runners-machine}

A runner connects out to XTM One and pulls the commands sent to its machine. XTM One never opens a port on the machine, and there is nothing inbound to allow in a firewall. Every command is recorded on the platform with its result.

## Before you start

- Runners are on by default. A platform engineer can remove them with `RUNNERS_ENABLED=false`, or cap the machines per user and the commands running at once on one machine: see the [Runners and XTM One Desktop section of the configuration reference](/reference/xtm-one/configuration/#runners-and-xtm-one-desktop).
- Everything starts on the **XTM One Desktop** page: open the user menu and choose **Get XTM One Desktop** (**My machines** inside the app).
- The agent that will use the machine needs the **Computer Use (Runners)** built-in tool, and agents need the Enterprise Edition with AI switched on.

## Enroll a machine

:::steps
1. On the **XTM One Desktop** page, under **Connect a machine**, click **New enrollment key**.

   ::screenshot{id=xtm-one-build-runners-desktop alt="The Connect a machine form of the XTM One Desktop page with the key name SOC jump host: the Desktop (recommended), Locked down and Custom permissions, and the Shell commands need approval and Reusable fleet key switches."}
2. Enter a **Key name**, and choose the permissions the machine starts with:
   - **Desktop (recommended)**: every capability, for the machine you sit at. Turn on **Shell commands need approval** to confirm each command in the chat.
   - **Locked down**: screenshots and browser control only. Enable more later on the machine's card.
   - **Custom**: choose the mode, the capabilities and the approval gate yourself, for servers and fleets.
3. Turn on **Reusable fleet key** to enroll several machines with one key; otherwise the key expires after one machine enrolls.
4. Click **Create key**. The key is shown only once: copy it with this instance's URL.
5. On a workstation, download XTM One Desktop from the same page (**Download for** your system), install it, and paste the URL and the key into its connect screen. On a server, copy the command under **Servers and headless machines (command line)**, put the key in it, and run it on the machine.
:::

The machine then appears under **Your machines**, online. It stores a durable credential in its system keychain and stays connected across reboots: you never paste the key again. Each enrolled machine is one runner you can name and label. Reinstalling on the same machine updates its record instead of creating a second one.

**Your machines** lists every machine you can see: yours, the ones shared with you and the company-managed ones. Each shows as a card by default, sorted by name; inside XTM One Desktop, the machine you are using comes first, marked **This machine**. To find one among many, search by name, hostname, label or tag, and filter by **Status** (**Online**, **Offline**, **Revoked**), **Platform** or **Permission mode**. **List view** compares the machines in a table (**Name**, **Status**, **Platform**, **Hostname**, **Permission mode**, **Commands**, **Last seen**) that **Export CSV** downloads; click a row to open the machine's page, its card expanded. The view you choose is kept in your browser.

On macOS, the system asks for **Screen Recording** and **Accessibility** permissions the first time an agent captures the screen or controls input. Screenshots, mouse and keyboard need a signed-in desktop session; shell commands, files and browser control also work on a headless server.

## Decide what agents may do on the machine

Each machine has its own permissions, on its card of the **XTM One Desktop** page (**Edit permissions**).

<!-- shot: xtm-one-build-runners-permissions -->

- **Permission mode**:
  - **Restricted**, the default: only the capabilities you enable run, **Screenshots**, **Mouse & Keyboard**, **Shell Commands**, **File Operations** and **Browser Control**. Enabling a capability is the consent: it then runs without asking.
  - **Full control**: every capability and every command, with no lists and no approval. Keep it for machines you fully trust for automation.
- For shell commands, in **Restricted** mode:
  - **Allowed shell commands**: a command that matches one of them runs at once. With approval off, only these commands run at all.
  - **Blocked shell commands**: a command containing one of these words is always refused. The block list wins over the allow list.
  - **Require approval for shell commands**: any other command waits for a person, in the chat when it was asked from XTM One Desktop, otherwise on the **XTM One Desktop** page. **Approve & always allow** and **Reject & always block** add the command to the matching list, so it never asks again.
- **Allowed paths**: file operations stay under these folders. Empty means no path restriction.

Revoking a machine (**Revoke runner**) cuts it off at once: its credential stops working, its queued commands end and its connection closes. Revoke a machine as soon as it is decommissioned. Uninstalling the app does not revoke it.

## Give an agent its machines

:::steps
1. Open the agent, its **Configuration** tab, then **Tooling**.
2. Under **Built-in Tools**, add **Computer Use (Runners)**, and save.
3. In the **Runners** card, click **Edit**, then select **Specific machines**, add **Labels**, or both. Leave both empty to let the agent use any machine you can access.
4. Click **Save**.
:::

An assignment of the agent can narrow that selection, never widen it. When more than one eligible machine is online, the agent must name the machine for any command that changes something, so a command never lands on the wrong one. In a chat opened in XTM One Desktop, the agent's commands go to the machine you are typing on unless you name another.

Each command shows in the conversation as a card: a shell command with its exit code and output, a file with a preview, a screenshot you can enlarge. Screenshots and files come back as files of the conversation, ready for an evidence workflow.

To let an agent drive a browser you signed in to, start Chrome or Edge with remote debugging on port 9222 and a profile of its own (`--remote-debugging-port=9222 --user-data-dir=...`), and sign in there once. The address the runner connects to is in the app's settings.

## Follow and stop long commands

A shell command that takes a while, such as a long download, a server or a virtual machine booting, can run as a background job. The agent gets an answer at once with the first lines of output, and the command keeps running on the machine after the agent's turn. The agent then lists the machine's jobs, reads the output of one, or stops it.

Background jobs need a runner of version 1.261103.10467 or later. An older runner is not sent the command, and the agent says so. The runner updates itself from your instance when automatic updates are on; otherwise update it with Check for updates, in the connection panel or the menu of XTM One Desktop.

To stop a shell command from the chat, click **Stop** on its card while it runs, or while its background job runs. A background job asks first (**Stop this job?**). The machine ends the command and every process it started, and the output printed until then stays readable. The card then reads **Stopped by you**, or the name of the person who stopped it, and the agent is told not to run the command again. You can stop a command that runs for you, or any command on a machine you manage.

To see every job a machine holds, expand the machine's card on the **XTM One Desktop** page and read **Background jobs**. Running jobs come first, each with its command, its state (**Running**, **Stopped**, **Exited** with the exit code), the size of its output and when it started. Click **Stop** on a running job to end it.

The list is asked to the machine itself when the card opens, and again when you click **Refresh**: it is not refreshed on its own, and each read is a command recorded on the machine. When the machine is offline, busy, has **Shell Commands** off or does not answer within 10 seconds, the section says so instead. Only people who manage the machine see this section.

:::example[Let the triage agent check the SOC jump host]
At night, Paul Garnier often asks **Night shift triage** whether the SOC jump host `soc-jump-01` saw the logins named in an alert. Sophie Laurent enrolls the server and gives the agent access to it, and to nothing else.

On the **XTM One Desktop** page, **New enrollment key**:

- **Key name**: `SOC jump host`
- **Custom**, mode **Restricted**, capabilities **Shell Commands** and **File Operations**
- **Shell commands need approval**: on

Marc Dupuis runs the headless install command on `soc-jump-01` with the key. On the machine's card, Sophie adds the label `soc`, sets **Allowed shell commands** to `last`, `journalctl -u sshd`, **Blocked shell commands** to `rm -rf`, `shutdown`, and **Allowed paths** to `/var/log`.

On **Night shift triage**, she adds **Computer Use (Runners)** and, in the **Runners** card, the label `soc`. The agent now reads the SSH logins of the jump host when Paul asks, and any other command waits for Paul's approval.
:::

## What's next

- [Create an agent](/xtm-one/build/agents/)
- [Let agents sign in to web applications](/xtm-one/build/browser-profiles/), for a browser that runs on the platform rather than on your machine.
- [Govern how AI is used](/xtm-one/administer/ai-governance/)
