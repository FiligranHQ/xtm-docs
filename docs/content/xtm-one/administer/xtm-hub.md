---
title: Connect XTM One to the XTM Hub
description: Register XTM One with your organization on the XTM Hub, like OpenCTI and OpenAEV, keep the registration alive, and unregister it.
doc_type: how-to
products:
  - xtm-one
  - xtm-hub
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-27
---

XTM Hub is Filigran's place for the platforms of your organization, its trials and its ready-made content. Once registered, XTM One appears among your organization's platforms on the XTM Hub, beside OpenCTI and OpenAEV, with its edition, and its own sidebar leads to the XTM Hub. This page registers the platform, explains what keeps the registration alive, and unregisters it. Letting the XTM Hub decide who signs in is a separate setting: see [Let the XTM Hub decide](/xtm-one/administer/sso/#let-the-xtm-hub-decide).

::diagram{id=xtm-one-administer-xtm-hub}

## Before you start

- You are an administrator of XTM One. Registering, checking and forgetting the registration are reserved to administrators; every user sees its status.
- You have an account on the XTM Hub, in the organization to register the platform with.
- Your browser allows pop-ups for XTM One: the XTM Hub opens in a window of its own.
- Both your browser and the platform reach the XTM Hub: the browser opens its page, and the platform confirms the registration with it, then checks it every hour. The XTM Hub is `https://hub.filigran.io` unless the platform engineer sets another in `XTM_HUB_DEFAULT_URL` (see the [XTM One configuration](/reference/xtm-one/configuration/#xtm-hub-registration)).
- The Community and the Enterprise Edition both register.

## Register the platform

:::steps
1. Open :menu[Administration > Settings > Platform]. The **XTM Hub** card reads **Not registered**.

   ::screenshot{id=xtm-one-administer-settings-parameters alt="The Platform tab of Settings: the Instance ID a license is generated for, the XTM Hub card Not registered with its Register on the XTM Hub button, and the Enterprise License card."}

2. To register with another XTM Hub than the default one, type its address in **XTM Hub URL**: an `https` address, `http` only for an XTM Hub on the same machine. Empty, the platform registers with the default XTM Hub, whose address the field shows in grey.
3. Click **Register on the XTM Hub**. The XTM Hub opens in a new window, and the card shows **Waiting for the XTM Hub window...** with **Cancel**.
4. In the XTM Hub's window, log in and choose the organization to register the platform with.
5. The window closes by itself, and XTM One says **This platform is registered with the XTM Hub**. The card now reads **Registered**, with the address of the **XTM Hub**, the date it was **Registered** and its **Last check**.
:::

XTM One sends the XTM Hub the platform's identifier, address, name, version and edition. The XTM Hub answers with a platform token, which XTM One keeps only once the XTM Hub confirms it, encrypted and bound to that XTM Hub and to the platform's identifier. XTM One never shows, logs nor audits the token.

When the browser blocks the window, XTM One says **The browser blocked the XTM Hub window. Allow pop-ups for this site and try again.** When the XTM Hub cannot send the token back to the page, it shows it instead: unfold **The XTM Hub showed a platform token instead**, paste it in **Platform token** and click **Save**. Closing the XTM Hub's window, or **Cancel**, registers nothing.

## What the registration brings

| Where | What changes |
| --- | --- |
| The XTM Hub | XTM One appears among your organization's platforms, with its edition: Community or Enterprise. |
| The product switcher of XTM One's sidebar | Its **XTM Hub** entry opens the XTM Hub you registered with. Before the registration, the entry of an administrator says **Register on the XTM Hub** and leads to :menu[Administration > Settings > Platform]; the entry of other users opens the default XTM Hub. |
| The home dashboard | The XTM Hub pill of the overview, beside OpenCTI's, OpenAEV's and OpenCRQ's, gives the registration's health and opens the XTM Hub: see [Read the home dashboard](/xtm-one/use/dashboard/). |
| :menu[Administration > Audit logs] | Each registration, each check that changed the status, and each registration forgotten, as `xtm_hub_registration`. |

The registration gives no agent access to the XTM Hub. For agents to work with the XTM Hub's API, connect the XTM Hub integration with a personal API token: see [Connect agents to your tools with integrations](/xtm-one/build/integrations/).

## Keep the registration alive

Every hour, XTM One asks the XTM Hub whether it still knows the platform, and sends its edition again: a license installed or removed after the registration reaches the XTM Hub within the hour, without registering again. **Check now**, on the card, asks at once.

| Status | When | What to do |
| --- | --- | --- |
| **Registered** | The XTM Hub confirmed the registration at the last check. | Nothing. |
| **Connectivity lost** | The XTM Hub did not confirm it, or could not be reached. The registration is kept and checked again every hour. | Check that the platform reaches the XTM Hub, then **Check now**. |
| **Register again** | The platform's identifier changed since the registration: `PLATFORM_ID` set afterwards, or a database restored or copied to another platform. XTM One sends the XTM Hub nothing until it is registered again. | **Register on the XTM Hub**, as above. |
| **Not registered** | Never registered, or the registration was removed on the XTM Hub: XTM One then forgets it at the next check. | Register the platform. |

## Unregister the platform

:::steps
1. Open :menu[Administration > Settings > Platform] and click **Unregister** on the **XTM Hub** card. The XTM Hub opens in a new window.
2. Confirm in the XTM Hub's window. It closes, and XTM One says **This platform is no longer registered with the XTM Hub**. The card reads **Not registered**.
:::

When the XTM Hub cannot be reached, **Forget locally** stops the checks and deletes the token on XTM One's side, after **Forget the XTM Hub registration?**: click **Forget**. The registration stays on the XTM Hub: remove it there with **Unregister** once the XTM Hub is reachable again, or register the platform again.

## A platform of an XTM Hub trial

A platform the XTM Hub deployed for a trial is registered without an administrator: its deployer registers it with the default XTM Hub, under the platform token the XTM Hub issued for it, and under the identifier it reported, which `PLATFORM_ID` must keep. While its trial license is valid, the card shows the default XTM Hub without letting you change it, and offers neither **Unregister** nor **Forget locally**: the trial's platform stays on the XTM Hub, as an OpenCTI of a trial does.

:::example[Register Grand Meridian's XTM One with its XTM Hub organization]
Hélène Marchetti, administrator of `https://xtm-one.grand-meridian.example`, wants the platform listed beside Grand Meridian's OpenCTI on the XTM Hub. She opens :menu[Administration > Settings > Platform], leaves **XTM Hub URL** empty, and clicks **Register on the XTM Hub**. In the XTM Hub's window, she logs in with her Grand Meridian account and chooses the organization `Grand Meridian Hotels`. The window closes, and the card reads **Registered**, with `https://hub.filigran.io` and today's date.

Two weeks later, Marc Dupuis installs XTM One's Enterprise license. Within the hour, the hourly check tells the XTM Hub, which shows the platform in Enterprise Edition: nobody registers it again.
:::

## What's next

- [Let the XTM Hub decide who signs in](/xtm-one/administer/sso/#let-the-xtm-hub-decide)
- [Set up the platform in Settings](/xtm-one/administer/settings/#platform)
- [What each edition of XTM One brings](/xtm-one/administer/editions/)
