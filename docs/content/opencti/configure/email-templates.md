---
title: Customize the emails OpenCTI sends
description: Write email templates for administrative messages to OpenCTI users, such as onboarding or maintenance notices, and send them on creation, in bulk or one by one.
sidebar_position: 10
doc_type: how-to
products:
  - opencti
edition: ee
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/email-templates/
---

An email template is a one-time message to OpenCTI users: a welcome email with the platform address and, when needed, an API token, a notice about a process change, a warning before maintenance. This page shows how to write a template, test it, and send it to new users, to a selection of users or to one user. Templates are not used for knowledge notifications or for disseminating reports: those go through [notifiers](/opencti/configure/notifiers/) and dissemination lists.

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Manage credentials** capability.
- The platform can send emails: see [Send emails from OpenCTI](/opencti/administer/smtp/).

OpenCTI comes with one template, "Built-In Template For Onboarding", that you can adapt rather than start from scratch.

## Create a template

:::steps

1. Open :menu[Settings > Security > Email templates] and click **Create Email template**. The **Create a template** panel opens.
2. Fill in the fields, then click **Create**. The template opens in the **Template Editor**.

   | Field | What to enter |
   | --- | --- |
   | **Name** | How you find the template in lists |
   | **Description** | Optional: when to use it |
   | **Email subject** | The subject line users see |
   | **Display name of the sender** | The sender name shown next to the platform's sender address |

3. Write the body in the editor. Format the text and insert images as in any rich text editor.
4. Add variables from the side panel, which lists them under **Attributes of the user** and **Attributes of the platform**: click **Copy attribute name to clipboard** next to a variable and paste it where the value should appear.
5. Click **Save template**. The tab shows **You have unsaved changes** until you do.

   ::screenshot{id=opencti-email-template-editor alt="The Built-In Template For Onboarding open in the Template Editor, with the body on the left and the side panel listing the attributes of the user and of the platform."}

:::

## Variables

| Variable | Replaced by |
| --- | --- |
| `$user.firstname` | First name of the recipient |
| `$user.lastname` | Last name |
| `$user.name` | User name |
| `$user.user_email` | Email address |
| `$user.account_status` | Account status, such as Active |
| `$user.objectOrganization` | The organizations the user belongs to, separated by commas |
| `$user.account_lock_after_date` | The date the account expires, empty if none |
| `$user.api_token` | A new API token, created for the recipient when the email is sent |
| `$settings.platform_url` | The URL of the platform |

:::warning
Each email that contains `$user.api_token` creates a new token, named `Template generated token`, for its recipient, test emails included. Use it only in onboarding emails, remind users to keep it secret, and revoke the tokens your tests create. The side panel lists this variable as `$user.token_user`; write `$user.api_token`, which is the name OpenCTI replaces.
:::

:::example[Welcome analysts from member hotel chains]
Example CERT opens its platform to the security teams of the hotel chains it serves. Each new account should get a welcome email that says where to log in and which organization the account belongs to.

- **Name**: `Member onboarding`
- **Description**: `Sent when an account is created for a member hotel chain`
- **Email subject**: `Your access to the Example CERT threat intelligence platform`
- **Display name of the sender**: `Example CERT`

The body, with the variables pasted from the side panel:

```text
Hello $user.firstname,

Your account on the Example CERT threat intelligence platform is ready.

Log in at $settings.platform_url with the address $user.user_email.
Your account belongs to: $user.objectOrganization.

Start with the reports labelled hotel-sector, such as the latest analyses of
the Velvet Harbor intrusion set. Questions go to cti@example.com.

The Example CERT team
```

For Leo Martin of Grand Meridian Hotels, created with the template chosen in **Email template**, the email reads "Hello Leo," and gives the platform address and "Grand Meridian Hotels". The template leaves out `$user.api_token` on purpose: members log in through the interface and need no API token.
:::

## Test a template

Click the **Send test email** button in the template's header. OpenCTI sends the email to your own address, with the variables filled with your account's values, so you see it as a recipient will.

## Change or delete a template

Open the template from the list. In the menu of its header, **Update** opens **Update a template**, where you change the name, the description, the subject and the sender name; the body stays in the **Template Editor**, saved with **Save template**. **Delete** removes the template.

## Send a template

| To | Do this |
| --- | --- |
| A new user, when you create the account | In the user creation form of :menu[Settings > Security > Users], choose the template in **Email template** |
| Several users | In :menu[Settings > Security > Users], select the users, click **Send email** in the toolbar, choose the template and click **Send** |
| One user | Open the user, click **Send email**, choose the template and click **Send** |

A playbook can also send a template, with the **Send email from template** component: see [Create a playbook](/opencti/use/automate/playbooks/create/). Every email sent is recorded in the [activity events](/opencti/configure/activity/events/).

Organization administrators can send a template from the page of a user they manage, but only users with **Manage credentials** see the **Email templates** menu and create, change or delete templates.

## Rendering limits

OpenCTI fills the variables of a template in an isolated worker thread, with a memory limit and a 5-second timeout, so that a faulty template cannot slow down the platform. A template that exceeds them fails to render, and the email, test emails included, is not sent. Notifier templates and the platform's own emails go through the same workers.

::::only{opencti=">=7.261030"}
The limits are configuration keys. Raise them, then restart the platform, only if large templates or large notification digests fail to render:

| Key (environment variable) | Default | Effect |
| --- | --- | --- |
| `safe_ejs:idle_pool_size` (`SAFE_EJS__IDLE_POOL_SIZE`) | `2` | Workers kept ready between renders. A render beyond them starts a new worker |
| `safe_ejs:max_renders_per_worker` (`SAFE_EJS__MAX_RENDERS_PER_WORKER`) | `1000` | Renders after which a worker is replaced by a fresh one |
| `safe_ejs:resource_limits:max_old_generation_size_mb` (`SAFE_EJS__RESOURCE_LIMITS__MAX_OLD_GENERATION_SIZE_MB`) | `50` | Main memory of a worker, in MB |
| `safe_ejs:resource_limits:max_young_generation_size_mb` (`SAFE_EJS__RESOURCE_LIMITS__MAX_YOUNG_GENERATION_SIZE_MB`) | `10` | Memory for short-lived objects, in MB |
| `safe_ejs:resource_limits:code_range_size_mb` (`SAFE_EJS__RESOURCE_LIMITS__CODE_RANGE_SIZE_MB`) | `10` | Memory for compiled code, in MB |
| `safe_ejs:resource_limits:stack_size_mb` (`SAFE_EJS__RESOURCE_LIMITS__STACK_SIZE_MB`) | `4` | Stack size, in MB |

They are in the [Technical customization](/reference/opencti/configuration/#technical-customization) section of the configuration reference.
::::

:::only{opencti="<7.261030"}
In this version, the limits are fixed: 50 MB of main memory, 10 MB for short-lived objects, 10 MB for compiled code and a 4 MB stack per render.
:::

## What's next

- [Access control](/opencti/administer/access-control/) to create users and organize them in groups.
- [Notifiers](/opencti/configure/notifiers/) for notifications about knowledge.
