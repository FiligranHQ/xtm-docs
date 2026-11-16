---
title: Microsoft Teams notifier samples
description: The two Microsoft Teams webhook notifiers OpenCTI ships with, one for live triggers and one for digests, their templates and how to adapt them.
sidebar_position: 1
doc_type: reference
products:
  - opencti
personas: [admin, developer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/notifier-samples/
---

OpenCTI creates two sample notifiers when the platform is first installed, in :menu[Settings > Customization > Notifiers]. Both use the Generic webhook connector to post a card to a Microsoft Teams channel. They do nothing until you give them the webhook URL of your channel. This page gives their templates and what to change; how notifiers and templates work is in [Send notifications by email or webhook](/opencti/configure/notifiers/).

| Notifier | Use it with | Posts |
| --- | --- | --- |
| Sample of Microsoft Teams message for live trigger | Live triggers | One card per event, with the operation, the date, the event message and a **See in OpenCTI** button |
| Sample of Microsoft Teams message for digest trigger | Digests | One adaptive card with the name and date of the digest, then one linked line per event |

Both are set to `POST` to `https://YOUR_DOMAIN.webhook.office.com/YOUR_ENDPOINT`.

## Adapt a sample

:::steps

1. Create an incoming webhook in the Teams channel that should receive the notifications, following the [Microsoft documentation](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook), and copy its URL.
2. In :menu[Settings > Customization > Notifiers], open the menu of the sample and choose **Update**. The **Notifier edition** panel opens.
3. Replace the `url` with the URL of your webhook.
4. In the `template`, replace the hard-coded platform address of the links with `<%= platform_uri %>`: `https://YOUR_OPENCTI_URL` in the live sample, `https://localhost:3000` in the digest sample.
5. In **Accessible for**, add the users or groups that may pick this notifier in their triggers. Keep the list short: the card is posted once per recipient of each trigger that uses the notifier, so one trigger per channel, owned by one user, avoids duplicate cards. See [Choose a connector](/opencti/configure/notifiers/#choose-a-connector).
6. Click **Test**. In the **Testing notifier** dialog, choose **Sample Notification** for the live sample or **Sample Digest** for the digest sample in **Choose target**, click **Test**, and check the card in Teams.
7. Click **Save**.

:::

::screenshot{id=opencti-notifier-teams-sample alt="The Notifier edition panel of a Microsoft Teams sample, showing the Generic webhook connector, the POST verb, the webhook URL placeholder and the start of the template."}

To keep the samples as models, duplicate their settings into a new notifier instead: create a notifier with the Generic webhook connector and paste the template.

## Live trigger template

The template of **Sample of Microsoft Teams message for live trigger**, as shipped. A live trigger sends one event per notification, so the template reads the first event only.

```ejs
{
  "type": "message",
  "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.thumbnail",
      "content": {
        "subtitle": "Operation : <%=notification_content[0].events[0].operation%>",
        "text": "<%=(new Date(notification.created)).toLocaleString()%>",
        "title": "<%=notification_content[0].events[0].message%>",
        "buttons": [
          {
            "type": "openUrl",
            "title": "See in OpenCTI",
            "value": "https://YOUR_OPENCTI_URL/dashboard/id/<%=notification_content[0].events[0].instance_id%>"
          }
        ]
      }
    }
  ]
}
```

## Digest template

The template of **Sample of Microsoft Teams message for digest trigger**, as shipped. A digest holds several groups of events, so the template loops over `notification_content` and over the events of each group, and adds a comma between items so that the result stays valid JSON.

```ejs
{
  "type": "message",
  "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.adaptive",
      "content": {
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "type": "AdaptiveCard",
        "version": "1.0",
        "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "<%=notification.name%>",
                "weight": "bolder",
                "size": "extraLarge"
              }, {
                "type": "TextBlock",
                "text": "<%=(new Date(notification.created)).toLocaleString()%>",
                "size": "medium"
              }
            ]
          },
          <% for(var i=0; i<notification_content.length; i++) { %>
          {
            "type": "Container",
            "items": [<% for(var j=0; j<notification_content[i].events.length; j++) { %>
              {
                "type" : "TextBlock",
                "text" : "[<%=notification_content[i].events[j].message%>](https://localhost:3000/dashboard/id/<%=notification_content[i].events[j].instance_id%>)"
              }<% if(j<(notification_content[i].events.length - 1)) {%>,<% } %>
            <% } %>]
          }<% if(i<(notification_content.length - 1)) {%>,<% } %>
          <% } %>
        ]
      }
    }
  ],
  "dataString": <%-JSON.stringify(notification)%>
}
```

`notification_content` is the same list as `content` in the [template variables](/opencti/configure/notifiers/#the-data-a-template-receives).

## What's next

- [Send notifications by email or webhook](/opencti/configure/notifiers/) to write your own templates.
- [Get alerted when knowledge changes](/opencti/use/automate/notifications/), where users attach a notifier to a trigger or a digest.
