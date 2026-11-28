---
title: Send intelligence by email
description: Send a finished intelligence document from an OpenCTI report or case to a list of recipients outside the platform, in blind copy, and keep the lists of recipients up to date.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
edition: ee
personas: [cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/dissemination-list/
---

A dissemination list is a named list of email addresses, such as the security contacts of your customers or of a partner CERT, who do not need an OpenCTI account. From the report or the case you wrote, you send its finished document, a PDF or an HTML file, to everyone on a list at once. The recipients are in blind copy, and every sending is logged. This page shows how to prepare the document under the right markings, send it, and keep the lists up to date.

## Before you start

- Your platform has an Enterprise Edition license.
- To send a document, your role has the **Disseminate files by email** capability.
- To create and change lists, your role has the **Manage dissemination lists** capability. Analysts who only send need a list created for them.
- The platform can send email: an administrator has set up the [SMTP configuration](/opencti/administer/smtp/).
- You are not in a draft: documents cannot be sent from a draft.

## Prepare the document under the right markings

OpenCTI sends the file you choose to every address on the list. It does not compare the markings of the file, or of what it contains, with the recipients: the markings you set when you generate the document are your control.

Generate the document from a fintel template, as described in [Export knowledge to a file](/opencti/use/share/export/#generate-a-finished-intelligence-document), and set **Content max marking definition levels** to the highest marking the recipients may receive. With TLP:GREEN, the indicators, threats and other objects marked TLP:AMBER or TLP:RED are left out of the document. The document lands in the **Content** tab of the container, where you can reread it before you send it.

## Send a document to a list

:::steps

1. Open the report or case and go to its **Content** tab.
2. In the list of files, click the **Disseminate** button (the paper plane icon) next to the PDF or HTML document. The **Disseminate a file** window opens.
3. Keep **Use OpenCTI template** on to send the email with the look of your platform, or turn it off for a plain email.
4. Choose the **Dissemination list**.
5. Enter the **Email subject**. Put the TLP of the document in it, so that recipients see it before they open the attachment.
6. Write the **Email body**, in Markdown. For an HTML document, you can turn on **Use file content as email body** instead: the document becomes the body of the email, with no attachment.
7. Check the **File** to send, then click **Send**.

:::

The message **Email sent successfully** shows once the platform has handed the email to the mail server. Delivery can then take several minutes, depending on the size of the attachment and your mail server.

:::example[Send the Velvet Harbor advisory to the hotel-sector ISAC]
Your team serves the hotel chains of the hospitality Information Sharing and Analysis Center (ISAC), whose sharing agreement stops at TLP:GREEN. The report "Velvet Harbor targets hotel booking systems" is ready.

1. From the report, generate a PDF from your advisory template with **Content max marking definition levels** at **TLP:GREEN** and **File marking definition levels** at **TLP:GREEN**. The intrusion set Velvet Harbor, marked TLP:AMBER, stays out of the document; the indicators to block, marked TLP:GREEN, are in it.
2. In the **Content** tab, click **Disseminate** next to the PDF and fill in:
   - **Use OpenCTI template**: on
   - **Dissemination list**: `Hotel sector ISAC`
   - **Email subject**: `[TLP:GREEN] Velvet Harbor targets hotel booking systems`
   - **Email body**: `Velvet Harbor phishes front-desk staff with fake booking requests, then exports guest records from the booking platform at night. The attached advisory lists the indicators to block and the actions to take.`
3. Click **Send**.

Each member of the ISAC receives the PDF in blind copy, and you receive a copy too. The sending appears in the activity of the platform with the report, the list and the file.
:::

## How the email is sent

- **Recipients are in blind copy.** No recipient sees the others. The visible recipient is a no-reply address, `no-reply@filigran.io` by default.
- **You get a copy.** Your own address is added in blind copy, so you see what the recipients received.
- **The sender** is the platform title with the sender address of the [SMTP configuration](/opencti/administer/smtp/).
- **Every sending is logged** in the activity of the platform, with the list, the container and the files sent.

## Create and maintain lists

:::steps

1. Open :menu[Settings > Security > Dissemination list] and click **Create Dissemination List**.
2. Enter a **Name** that tells analysts who is on the list and, if you want, a **Description**.
3. In **Emails (1 / line)**, paste the addresses, one per line.

   ::screenshot{id=opencti-share-dissemination-list-create alt="The Create a dissemination list panel, with the Name and Description fields and the Emails (1 / line) field where each address goes on its own line."}

4. Click **Create**. The list shows with its **Number of recipients**.

:::

The form checks each line: a line that is not a valid email address, or an empty line, shows **Each line must contain a valid email address**. A list holds up to 500 addresses.

To change a list, such as when a member joins or leaves, open its actions menu (⋮) and choose **Update**. The **Update a dissemination list** panel has the same fields as the creation. A change applies to the next sending; emails already sent are not affected.

:::example[Create the list of the hotel-sector ISAC]
In **Create a dissemination list**:

- **Name**: `Hotel sector ISAC`
- **Description**: `Security contacts of the member hotel chains of the hospitality ISAC. Sharing agreement up to TLP:GREEN.`
- **Emails (1 / line)**:

  ```text
  security@grand-meridian.example.com
  soc@harbor-hotels.example.com
  cti@seaside-resorts.example.com
  ```

The list shows with 3 in **Number of recipients**. Writing the sharing limit in the description reminds every analyst which markings the list may receive. When a new hotel chain joins the ISAC, add its address to the list rather than to each email.
:::

## Change the platform defaults

Two configuration keys of the platform, in [Functional customization](/reference/opencti/configuration/#functional-customization), change how dissemination works for everyone:

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:dissemination_list:to_email` | `APP__DISSEMINATION_LIST__TO_EMAIL` | `no-reply@filigran.io` | The visible recipient of every email |
| `app:dissemination_list:max_list_size` | `APP__DISSEMINATION_LIST__MAX_LIST_SIZE` | `500` | The number of addresses a list may hold |

The creation and update forms refuse more than 500 addresses whatever `max_list_size` says (**You cannot have more than 500 e-mail addresses**). A larger value only applies to lists created or changed through the API.

:::example[Show your team's address as the recipient]
Members of the ISAC should see an address of your team, not the default no-reply address, as the visible recipient. In the environment of the platform container:

```yaml
services:
  opencti:
    environment:
      - APP__DISSEMINATION_LIST__TO_EMAIL=cti-advisories@example.com
```

After a restart, every email shows `cti-advisories@example.com` as its recipient, while the members stay in blind copy. Make sure the mailbox exists: members who reply all write to it.
:::

## Check that it works

Your copy in blind copy is the proof that the email left: the message **Email sent successfully** also shows when email is turned off in the SMTP configuration, and nothing is sent then. If your copy does not arrive, ask an administrator to check the [SMTP configuration](/opencti/administer/smtp/), and look in the spam folder of a test recipient. A recipient address the mail server rejects is only recorded in the platform logs.

:::only{opencti=">=7.261127"}
When an administrator has required STARTTLS for a mail server OpenCTI logs in to, and that server does not upgrade the connection to TLS, nothing is sent: **Send** shows an error instead of **Email sent successfully**. Ask the administrator to check the encryption settings of the [SMTP configuration](/opencti/administer/smtp/#configure-the-mail-server).
:::

## What's next

- [Export knowledge to a file](/opencti/use/share/export/), to generate the finished intelligence document.
- [Brand your PDF reports](/opencti/configure/fintel-designs/), for the look of PDF documents.
- [Send emails from OpenCTI](/opencti/administer/smtp/), for the sender address.
