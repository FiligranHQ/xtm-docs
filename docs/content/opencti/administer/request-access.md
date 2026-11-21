---
title: Let users request access
description: Let OpenCTI users ask for knowledge they cannot see because of organization segregation, and set who validates or declines these requests.
sidebar_position: 24
doc_type: how-to
products:
  - opencti
edition: ee
personas: [admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/request-access/
---

With organization segregation, some knowledge exists in OpenCTI but is not shared with every organization. A user who tries to create it again gets an error, because it already exists out of their sight. Request access turns that dead end into a workflow: the user asks for the knowledge, a validator of the owning organization accepts or declines, and on acceptance the knowledge is shared with the user's organization. This page shows how to set it up and how requests are handled. Read [Separate data between organizations](/opencti/administer/organizations/) first if it is new to you.

## Before you start

- Your platform has an Enterprise Edition license.
- A platform main organization is set in [Set platform security policies](/opencti/administer/policies/): request access only works with organization segregation on.
- Your role has the **Manage customization** capability, to configure the Request for information entity type.
- A group exists for the people who will validate requests, for example the analysts of the platform main organization. Their role needs **Restrict organization access**.

## Set up the workflow

Requests are a special kind of request for information (RFI), with their own statuses.

:::steps

1. Open :menu[Settings > Customization > Entity types] and choose **Request for information**.

   :::only{opencti=">=7.260811"}
   Open its **Workflow** tab.
   :::

2. Next to **Specific Workflow for Request Access**, click the edit icon and add the statuses a request goes through. A new request takes the first status of the list.
3. Next to **Request access actions configuration**, click the edit icon. In **Request Access Configuration**, set:
   - **On approval move to status:** the status of an accepted request;
   - **On decline move to status:** the status of a declined request;
   - **Validator group membership:** the group whose members can validate or decline requests.
4. Click **Update**. The card now shows both statuses and the **Validator membership:** instead of **Disabled**.

:::

::screenshot{id=opencti-request-access-workflow alt="The Specific Workflow for Request Access card with the NEW, APPROVED and DECLINED statuses, and the Request access actions configuration card showing APPROVED on approval, DECLINED on decline and the Harbor analysts validator group."}

:::example[Let the analysts of Example CERT answer access requests]
Example CERT is the platform main organization, and its analysts, in the Harbor analysts group, decide what member hotels may see.

- **Specific Workflow for Request Access**: `NEW`, `APPROVED` and `DECLINED`, in this order, so that a new request starts as `NEW`.
- **On approval move to status:** `APPROVED`.
- **On decline move to status:** `DECLINED`.
- **Validator group membership:** Harbor analysts, whose role you extend with **Restrict organization access**.

Leo Martin, of Grand Meridian Hotels, then tries to create the intrusion set Velvet Harbor, which Example CERT has not shared with the hotel. He clicks **Request Access** and writes `Our web proxy saw booking-velvetharbor.example; we need the related indicators`. Lena Ortiz, of Example CERT and Harbor analysts, opens the request and clicks **Validate**: Velvet Harbor is shared with Grand Meridian Hotels, and Leo is notified.
:::

## How a request goes

:::steps

1. A user creates an entity that already exists but is not shared with their organization. The error says "Restricted entity already exists, you can request access", with a **Request Access** button.
2. The user clicks it, explains why they need the knowledge, chooses one of their organizations, and sends the request.
3. OpenCTI creates a request for information that contains the requested knowledge and carries its markings. The request is restricted to the members of the validator group who belong to an organization the knowledge is shared with, or to the platform main organization when it is shared with none. Each of them gets a notification, through their own notifiers, as soon as the request is created.
4. A validator opens the request and, in the **Processing status** card, clicks **Validate** or **Decline**. On validation, the knowledge is shared with the requesting organization. The request moves to the configured status, and the requester is notified of the result.

:::

To validate or decline, a user must be able to edit the request and have the **Restrict organization access** capability, which allows sharing with organizations.

## Limits

- Knowledge restricted with authorized members cannot be requested: only organization sharing can be requested. See [Restrict one object to chosen people](/opencti/administer/sharing/).
- Requests cannot be made or answered from inside a draft.
- The **Request Access** button only shows when the whole configuration is in place: Enterprise Edition, a platform main organization, both statuses and a validator group.
- Once configured, request access cannot be turned off from the form. Clearing **Validator group membership:** leaves the feature on with no valid validator: new requests then reach only the users who bypass all capabilities. Change the validator group instead of clearing it.

## What's next

- [Separate data between organizations](/opencti/administer/organizations/), to share knowledge with organizations.
- [Restrict one object to chosen people](/opencti/administer/sharing/), for restrictions on single objects.
- [Configure entity types](/opencti/configure/entities/), for the other workflow settings.
