---
title: Let agents sign in to web applications
description: Save a web application and its sign-in as a browser profile, sign it in once, and let an agent's browser open it signed in without ever seeing the credentials.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-10
migrated_from:
  - xtm-one:/usage/resources/browser-profiles/
---

A browser profile is a web application and its sign-in, kept by XTM One: the address to open, the account to use, and the session that sign-in saved. Give a profile to an agent and its **Browser** tool opens the application signed in, to read a dashboard, check a ticket or collect what a report needs, while the agent never sees the password. This page creates a profile, signs it in, gives it to an agent, and covers multi-factor authentication and private networks.

::diagram{id=xtm-one-build-browser-profiles-signin}

## Before you start

- Browser profiles are not an AI feature: creating one and signing it in work with AI switched off. The agent that uses it needs the **Browser** built-in tool (see [Write custom tools](/xtm-one/build/tools/#the-built-in-tools)).
- Only an administrator can make a profile **Company-wide**.
- Profiles reach public addresses only. For an application on a private network, an administrator turns on **Browser profiles on private networks** in :menu[Administration > Settings > Access and security > Security options] (or sets `BROWSER_PRIVATE_TARGETS`, see [Browser profiles](/reference/xtm-one/configuration/#browser-profiles) in the configuration reference), and the profile lists the hosts it may reach. Loopback, link-local and cloud metadata addresses stay refused either way.

## Create a profile

:::steps
1. Open :menu[Tools & Integrations > Connections]: it opens on the **Browser profiles** tab. Click **New browser profile**.
2. Fill **Name**, then **Start URL**: the application's home page, where every session opens and is checked.
3. Fill **Sign-in URL (optional)** when the sign-in starts on another page, such as the page the application's single sign-on button opens.
4. Under **Sign-in credentials**, fill **Username**, **Password** and **TOTP secret (base32 or otpauth:// URI)**. They are stored encrypted and never shown again: a blank field later keeps what is stored, **Clear** removes it. With a secret vault granted, **From a vault** reads a credential from the vault at each sign-in instead, so a password rotated there is followed (see [Secret vaults](/xtm-one/administer/secret-vaults/)).
5. For a private application, list its hosts in **Private-network hosts (optional)**.
6. Choose **Keep the session alive**: how often XTM One refreshes the saved session so it does not expire between uses, from **Never** to **Every day**. A new profile starts at **Every 4 hours**.
7. As an administrator, turn on **Company-wide** to let everyone use the profile, with their agents, without seeing its credentials.
8. Click **Save**.
:::

::screenshot{id=xtm-one-build-browser-profiles-new alt="The New browser profile dialog filled for the booking back office: the start and sign-in URLs, the service account in Username, the empty Password and TOTP secret fields, and Keep the session alive set to Every 2 hours."}

Credentials are typed on this page only. An agent can create a profile and set its addresses from a chat, but never takes a credential in a conversation: it asks you to add them here.

## Sign it in

Click **Sign in** on the profile's card. The sign-in runs in XTM One's own browser, in the profile's session: signing in to the application in a tab of your browser does not sign the profile in.

- **Single sign-on**: an application that sends you to an identity provider (SAML or OpenID Connect) signs in like a form. The sign-in follows the whole way back and succeeds only when the start URL shows the signed-in application.
- **Multi-factor authentication**: with the account's TOTP secret, the sign-in answers the code prompt by itself. Without it, the sign-in pauses on the prompt for 5 minutes: type the code on the card, or give it to the agent in the chat. For a push approval, the card shows the number to enter in your authenticator app, and the sign-in continues once you approve.
- **Passkeys and security keys** cannot be used by a server's browser: the sign-in chooses another method, an authenticator code when the profile has the TOTP secret.

<!-- shot: xtm-one-build-browser-profiles-card -->

The card then reads **Signed in**. When the application could not be checked after the sign-in, the session is saved anyway: the message reads **Signed in, not checked** and the card **Saved, not checked**. A profile XTM One can sign in again with no person shows **Renews by itself**: XTM One signs it in again when the session expires, so agents keep working unattended. That is a profile with the password and a valid TOTP secret, a profile whose password signs in without asking a person for anything, or a saved identity-provider session. New credentials count as renewing by themselves until a sign-in has to ask a person for a code or an approval. **Check the session** checks it now; **Test sign-in** runs the whole sign-in once without saving anything and shows the steps it took, to see where an unusual sign-in stalls. **Sign-in selectors and timeouts (unusual sign-ins only)** holds the settings for applications whose sign-in the defaults do not follow. When the application itself does not answer (it restarts, or refuses with an HTTP 5xx or 429), the card reads **Saved, not checked** rather than **Session expired**, and nothing signs in again until it answers.

## Give it to an agent

On the agent's page, open **Configuration**, then the **Tooling** section. With the **Browser** tool on, the **Browser profiles** card lists the profiles the agent may sign in with: click **Edit**, select the profile and **Save**. An agent never reaches a login it was not given.

- Turn on **Read-only** for the binding when the agent only needs to read: it can open pages, read their text and take screenshots, never click, type or run a script. A page reached by a link can still change something in an application that acts on a simple visit, so give such applications to agents you trust.
- A run opens a profile only for a person who may use it: its owner, the people and groups it is shared with, or everyone for a company-wide profile. An administrator's role does not widen that, and a run that acts for nobody opens none. See [Share agents and resources](/xtm-one/build/sharing/).
- When the agent's session closes, or after five idle minutes, XTM One saves the refreshed session if it still opens the application.

Every time an agent opens a profile, the audit log records it with the agent and the person it acted for. Your own General Assistant uses the profiles you choose under :menu[My profile > General Assistant Tools], once its **Browser** tool is on.

:::example[Let the night shift agent read the booking back office]
Paul Garnier, on the SOC night shift, checks whether a guest record was viewed from an unusual account, as Velvet Harbor did on 9 January 2026. Sophie Laurent, the AI engineer, gives **Night shift triage** read access to the booking back office:

| Field | Value |
| --- | --- |
| **Name** | `Booking back office` |
| **Start URL** | `https://backoffice.grand-meridian.example/reservations` |
| **Sign-in URL (optional)** | `https://backoffice.grand-meridian.example/sso/start` |
| **Username** | `svc-xtm-night@grand-meridian.example`, a read-only service account |
| **Password**, **TOTP secret** | **From a vault**: the item `xtm-one/booking-back-office` of the vault Hélène Marchetti granted |
| **Keep the session alive** | **Every 2 hours** |
| **Company-wide** | Off: shared with the **SOC** group |

She clicks **Sign in**; the card reads **Signed in** and **Renews by itself**. In the **Browser profiles** card of **Night shift triage**, she selects it with **Read-only** on. At 03:00, Paul asks the agent who opened the record of the guest named in an alert: the agent opens the back office signed in, reads the access history, and answers, without ever holding the password.
:::

## Check that it works

The card reads **Signed in**. In a chat with the agent, ask for something only the signed-in application shows: the agent's browser opens it without a sign-in page. If the session expired and the profile cannot renew by itself, the agent says so: sign in again on the card, then ask again.

## What's next

- [Write custom tools](/xtm-one/build/tools/), for an application with an API the agent can call directly.
- [Reach on-premise tools through a relay](/xtm-one/build/relays/).
- [Secret vaults](/xtm-one/administer/secret-vaults/).
