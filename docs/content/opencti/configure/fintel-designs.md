---
title: Brand your PDF reports
description: Give the PDF reports your team sends to customers and partners your own logo and colors with a fintel design, make it the default, and apply it when you export.
sidebar_position: 6
doc_type: how-to
products:
  - opencti
edition: ee
personas: [cti-lead, admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/fintel-designs/
---

A fintel design gives the finished intelligence (FINTEL) documents your team sends out your own look: your logo and your colors instead of OpenCTI's. You create it once, make it the default, and every PDF an analyst exports from a report carries your brand without anyone choosing it. This page shows how to create a design, check it on real data, make it the default, and apply it to an export. What the document says comes from a fintel template, written per entity type in [Customize entity types](/opencti/configure/entities/#create-fintel-templates); the design only dresses it.

## What a design changes

A design sets four things, and only these: the content, the sections and the markings of the document come from the template and the container.

| Field | Where it shows in the PDF | When left empty |
| --- | --- | --- |
| **Logo (png, jpg, svg or webp)** | The cover page and the back page | The white OpenCTI logo |
| **Background primary color** | The cover page, which fades from this color to the secondary one, and the thin band at the top of every inner page | Near black, `#00020C` |
| **Background secondary color** | The end of that fade | Deep blue, `#0015A8` |
| **Text color** | The title and the text written on the cover | White |

Choose a logo and a text color that stay readable on both background colors: the logo is often a white or light version of your brand mark.

:::only{opencti=">=7.261127"}
The page format follows the widest table of the template. A table of 8 to 11 columns puts the whole document in landscape A4, and a table of 12 columns or more in landscape A3, with smaller text in the table. The footer of every page but the back page gives the **File marking definition levels** chosen at export and the page number, on every format.
:::

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Manage customization** capability, to create and change designs. Deleting a design also needs **Delete knowledge**. Analysts only need to export: they pick designs without these capabilities.
- Your logo is a PNG, JPG, SVG or WebP file, and you know your colors as hexadecimal codes, such as `#1A2B3C`.

## Create a design

:::steps

1. Open :menu[Settings > Customization > Fintel design] and click **Create Fintel Design**. The **Create a fintel design** panel opens.
2. Enter a **Name** and a **Description** that tell analysts which documents the design is for, then click **Create**. The design opens, with the OpenCTI logo and colors in the **Preview**.
3. In the **Configuration** card, upload your logo in **Logo (png, jpg, svg or webp)**.
4. Set **Background primary color**, **Background secondary color** and **Text color**. Each change is saved and shown in the **Preview** at once.

   ::screenshot{id=opencti-fintel-design-editor alt="A fintel design open on its Configuration card, with a logo and three colors set, next to the Preview of the cover page in those colors."}

:::

A background color must be a hexadecimal code of three or six digits after `#`. Anything else shows **Invalid hexa color code** under the field and is not saved. To change the name or the description later, click **Update**: the **Update a Fintel design** panel opens.

:::example[Brand the reports sent to hotel members]
Example CERT sends its hotel members monthly PDF reports on Velvet Harbor. The team lead wants them to carry the logo and colors of the Harbor CTI program, so that members recognize them at a glance, and does not want each analyst to remember a setting.

- **Name**: `Harbor CTI brand`
- **Description**: `Cover and colors for reports sent to hotel-sector members`
- **Logo (png, jpg, svg or webp)**: `harbor-cti-logo.png`, the white version of the program's mark
- **Background primary color**: `#0B3D5C`, the dark blue of the program
- **Background secondary color**: `#1F7A8C`, the teal the cover fades to
- **Text color**: `#FFFFFF`, readable on both blues

The **Preview** shows a cover that fades from dark blue to teal, with the Harbor CTI logo and white text. Once the design is the default, an analyst who exports the report "Operation Night Desk targets European hotels" as PDF finds **Harbor CTI brand** already selected in **Fintel design**, and the PDF opens on that cover.
:::

:::only{opencti=">=7.260811"}
## Make a design the default

The default design is selected for you wherever a design can be chosen, so analysts get your brand without picking it each time. There is one default design at most.

- In the list, open the row menu of a design and choose **Set as default**. If another design is the default, confirm that it is replaced in **Replace default design?**.
- You can also turn on **Set as default** when you create or update a design.
- To go back to the OpenCTI look, choose **Remove default** on the current default design.

An analyst can still pick another design, or none, at export time. When no design is selected, the document uses OpenCTI's look.
:::

:::only{opencti="<7.260811"}
Your version has no default design: analysts pick the design each time they export. Tell them its name, and put it in the description of your fintel templates.
:::

## Check a design on real data

To see the design with real data before anyone exports, open a fintel template:

:::steps

1. Open :menu[Settings > Customization > Entity types] and choose the entity type, for example **Report**.
2. Open one of its fintel templates and select the **Template Preview** tab.
3. Choose an **Entity** to fill the template and a design in **Fintel design**. The preview shows the document as it will be exported.

:::

## Apply a design to a PDF export

Analysts apply a design when they export a container. They need the **Upload knowledge files**, **Download knowledge export** and **Generate knowledge export** capabilities.

::::only{opencti=">=7.261030"}
:::steps

1. Open the report, or any container, and select its **Content** tab.
2. Click **Generate an export based on a template**, in the files list of the tab. The **Generate an export** dialog opens with the **HTML content files to PDF** connector already chosen.
3. Keep **Export as fintel** on and choose the **Template**. OpenCTI fills the template with the container's data.
4. Check the design in **Fintel design**. When a default design exists, it is already selected.
5. Under **Page options**, keep **Include cover page** on: the cover carries most of the design. Without it, the document only keeps the colored band at the top of each page and the logo on the back page.
6. Click **Create**. The PDF appears in the files of the container when it is ready.

:::

::screenshot{id=opencti-fintel-design-export alt="The Generate an export dialog of a report with the HTML content files to PDF connector, the template, the Harbor CTI brand design preselected in Fintel design, and the Page options."}
::::

::::only{opencti="<7.261030"}
:::steps

1. Generate the FINTEL file first: on the **Content** tab of the container, click **Generate an export based on a template** and choose the template. The file appears in the files of the tab.
2. Click **Generate an export** and choose **PDF** as the format.
3. Choose **HTML content files to PDF** as the **Connector**, and the FINTEL file in **File to export**.
4. Choose the design in **Fintel design**. When a default design exists, it is already selected.
5. Click **Create**. The PDF appears in the files of the container when it is ready.

:::
::::

## What's next

- [Export knowledge](/opencti/use/share/export/) for every export format and connector.
- [Customize entity types](/opencti/configure/entities/#create-fintel-templates) to write the fintel templates that your designs dress.
