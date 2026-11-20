---
title: Customize entity types
description: Set mandatory fields, default values, custom fields, confidence scales, references, hidden types, statuses and templates for each OpenCTI entity type.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/entities/
---

Each entity and relationship type of OpenCTI has its own settings page: which attributes are mandatory, which values creation forms fill in by default, whether every change must cite a source, whether the type shows in the interface, and how its overview looks. This page walks through those settings and what each one also does to the API and to connectors, so that the forms your analysts use match the way your team works without blocking the data that comes in.

## Before you start

- Your role has the **Manage customization** capability.
- The **Templates** tab needs an Enterprise Edition license.

  :::only{opencti=">=7.261109"}
  So do custom fields. Everything else works without it.
  :::

  :::only{opencti="<7.261109"}
  Everything else works without it.
  :::

## Open the settings of a type

::::steps

1. Open :menu[Settings > Customization > Entity types]. The list shows, for each type, whether it has a **Workflow status**, **Enforce references**, **Automatic references at file upload**, and whether it is **Hidden in interface**.
2. Click a type, for example **Report**.

   :::only{opencti=">=7.260811"}
   The page shows the **Visibility** and **References** cards, which apply to every type, then tabs for the settings this type supports: **Workflow**, **Attributes**, **Templates**, **Overview Layout** and **Custom Views**.
   :::

   :::only{opencti="<7.260811"}
   Every setting is on one page, in cards: **Configuration** (visibility and references), **FINTEL Templates** on container types, **Workflow**, **Attributes** and **Overview layout customization**. Where this page says "tab", use the card of the same name.
   :::

::::

A type only shows the tabs and options it supports. A greyed-out option says **This configuration is not available for this entity type** when you hover its information icon.

::screenshot{id=opencti-entities-report-settings alt="The settings page of the Report type, with the Visibility and References cards above the Workflow, Attributes, Templates, Overview Layout and Custom Views tabs."}

To change one of these options for several types at once, tick the types in the list and click the matching button of the toolbar: **Automatic references at file upload**, **Hidden in interface** or **Enforce references**. In the dialog, turn **Enable this feature** on or off, then click **Update**. The dialog names the selected types that do not support the option; they are left unchanged.

## Hide a type

Hide the types your team does not use, for example Narrative and Channel if you do not track disinformation. A hidden type disappears from menus and creation forms; existing objects of that type stay in the database.

:::steps

1. Open the type in :menu[Settings > Customization > Entity types].
2. Under **Hidden in interface**, turn on **Hide in the platform**.

:::

The same card shows where else the type is hidden: **Hidden in groups** and **Hidden in organizations**. To hide a type only for some users, edit their group in :menu[Settings > Security > Groups] or their organization in :menu[Settings > Security > Organizations] and fill **Hidden entity types**. The field of the same name in :menu[Settings > Parameters] lists every type hidden platform-wide. See [Platform parameters](/opencti/configure/parameters/).

## Require references

The **References** card has two options, both off by default:

| Option | What it does |
| --- | --- |
| **Create external reference at upload** | Each file uploaded to an object of this type also becomes an external reference of that object |
| **Enforce references** | An object of this type cannot be created without at least one external reference, nor changed without naming the external reference that justifies the change. In the interface, a change opens the **Reference modification** dialog, where the user picks the references and can add a **Message** |

**Enforce references** is checked by the platform, not only by the forms: it applies to the API too. Users whose role has the **Bypass enforced reference** capability skip it. The default Connector role has it, so connectors keep importing; an integration that runs with a user without it gets the error `You must provide at least one external reference for this type of entity/relationship`. The option slows down manual work, so keep it for the types where every change must be traced, such as reports and incidents.

Two platform settings complete these options:

- `app:reference_attachment` (`APP__REFERENCE_ATTACHMENT`), off by default: when on, every new external reference must come with an attached file, on every type. Users with **Bypass enforced reference** are exempt. See the [configuration reference](/reference/opencti/configuration/#maps-references).
- `app:enforce_references` (`APP__ENFORCE_REFERENCES`) is a legacy list of types. Only a 2023 migration reads it, to copy it into the **Enforce references** option of each type; setting it today changes nothing. See the [configuration reference](/reference/opencti/configuration/#functional-customization).

## Make attributes mandatory and set default values

The **Attributes** tab lists the attributes of the type, with a **Mandatory** column and a **Default value(s)** column. A greyed-out icon means the setting is fixed by the data model: an attribute that Structured Threat Information Expression (STIX) requires is always mandatory.

:::steps

1. Open the **Attributes** tab of the type.
2. Click an attribute. The **Update the attribute** panel opens.
3. Turn on **Mandatory** to make creation forms refuse an object without this attribute.
4. Fill **Default value** to prefill the creation forms.
5. Click **Update**.

:::

Mandatory attributes are checked on every creation, in forms and through the API, except for users whose role has **Bypass mandatory fields**. The default Connector role has it, so connectors keep creating objects without the attribute. For everyone else, set a default value when the attribute is often unknown at creation time.

::screenshot{id=opencti-entities-attribute-edition alt="The Update the attribute panel for the Report types attribute of reports, with the Mandatory switch and a Default value field set to threat-report."}

:::example[Route every report and trace every incident]
At Example CERT, a report without a type cannot be routed: the SOC reads threat reports, the hotel members read incident reports. Most reports the team writes are threat reports. The hotel members also ask that every change to an incident cite its source document.

1. In :menu[Settings > Customization > Entity types], open **Report**, then the **Attributes** tab.
2. Click **Report types**, turn on **Mandatory** and set **Default value** to `threat-report`. Click **Update**.
3. Back in the list, open **Incident** and, in the **References** card, turn on **Enforce references**.
4. Open **Narrative**, then **Channel**, and turn on **Hide in the platform** for both: the team does not track disinformation.
5. Ask the platform engineer to require a file on every new external reference:

```yaml
services:
  opencti:
    environment:
      APP__REFERENCE_ATTACHMENT: "true"   # every new external reference needs a file
```

An analyst who creates the report "Velvet Harbor infrastructure moves to new hosting" finds **Report types** marked with an asterisk and already set to threat-report; they change it to incident only for incident write-ups. A CSV import of reports without a type also gets threat-report. When the analyst changes the severity of an incident, OpenCTI asks for the external reference behind the change, and a new reference needs the hotel's report attached. The connectors, whose role bypasses both checks, keep importing. Narrative and Channel no longer appear in the menus.
:::

Default values apply wherever OpenCTI creates an object of this type without the attribute: creation forms, and CSV and JSON imports that use a mapper. A mapper can set its own default, which wins over the one set here. See [Map CSV files to knowledge](/opencti/configure/mappers/csv/#fill-missing-values).

A few attributes behave differently:

- **Markings**: there is no value to pick. **Activate/Deactivate default values** applies the default markings of the creator's group when an object is created without markings.
- **Authorized members**: setting a default needs an Enterprise Edition license.

## Adjust the confidence scale

Attributes with a scale, such as **Confidence**, have a **Scale configuration** section in the same panel. Pick a scale in **Selected scale template**, or change **Limits** and **Ticks** under **Customize scale** to name your own levels, then click **Update**. The scale changes how the levels read in forms and lists; a user still cannot set a confidence above their own maximum level. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

::::only{opencti=">=7.261109"}
## Add custom fields to an entity type

A custom field, created in [Manage labels, vocabularies and templates](/opencti/configure/taxonomies/#custom-fields), appears in the forms of a type once you add it here. Adding, changing and removing a custom field requires the Enterprise Edition; whether it is mandatory and its default value are set per type.

:::steps

1. Open the **Attributes** tab of the type. Under the attributes, the **Custom fields** card lists the custom fields of the type, with their **Label**, **Technical name**, **Type**, whether they are **Mandatory**, and their **Default value**.
2. Click **Add custom field**. In the **Add a custom field** dialog, pick the field in **Custom fields**, turn on **Mandatory** if every object of this type must have a value, then click **Add**.
3. To set a default value, open the menu of the field in the card, choose **Update**, fill **Default value**, then click **Update**. For a **Date** field, **Use current date/time (@now)** fills the creation date of each object instead of a fixed date.

:::

::screenshot{id=opencti-entities-custom-fields alt="The Custom fields card of the Attributes tab of the Report type, with its Add custom field button and the Member impact and Affected booking platform fields, their technical names, types, mandatory flags and default values."}

A mandatory custom field and a default value work like those of a built-in attribute: they apply to every creation, in forms, imports and the API, and users whose role has **Bypass mandatory fields** skip the mandatory check.

:::only{opencti=">=7.261117"}
OpenCTI checks a default value when you save it, so that every new object gets it. The default of an **Integer** field must be a whole number within the bounds of the field; the dialog shows **The value must be an integer** or **The value must be between min and max value** and keeps **Update** greyed out until you fix it. The default of a **Decimal** field can be any number, such as `0.75`. The platform refuses a value outside the options of a selection list, or a date it cannot read, in the API too.
:::

:::only{opencti="<7.261117"}
OpenCTI does not check a default value when you save it. Enter one the field accepts, such as a whole number within the bounds of an **Integer** field: otherwise new objects of the type do not get it, or their creation fails.
:::

**Remove**, in the menu of the field, takes it off this type: OpenCTI warns that **The values stored for this custom field will be permanently removed from all existing entities of this type.** Without the Enterprise Edition, the card stays, but a mandatory custom field can only be made optional.
::::

## Set up statuses

The **Workflow** tab sets the processing statuses an object of this type moves through, for example New, In progress and Analyzed. Statuses come from the status templates of :menu[Settings > Taxonomies > Status templates]. How to design and use them is in [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/); the draft workflow is in [Approve drafts step by step](/opencti/use/automate/draft-workflow/).

:::only{opencti=">=7.261030"}
The tab also has the **Entity status sync** switch, off by default. Turn it on when you receive this type from another OpenCTI platform through a [live stream](/opencti/use/ingest/streams/). Without it, OpenCTI keeps the internal reference of the remote status as it is, and that reference matches no local status. With it, OpenCTI applies the local status that has the same name:

- if a local status matches, the object gets it;
- if none matches, a new object gets the default status of its workflow, and an object already synchronized keeps its current status.

The option only affects objects received from a live stream. Both platforms need statuses with the same names for this type.
:::

In Enterprise Edition, the **Workflow** tab of some types also holds the settings of request access. See [Let users request access](/opencti/administer/request-access/).

## Create FINTEL templates

On container types, such as reports, groupings and cases, the **Templates** tab holds FINTEL templates: document models that turn a container into a finished intelligence file. Users pick a template when they generate a file from the container.

:::steps

1. Open the **Templates** tab of a container type and click **Create a new template**, the **+** of the **FINTEL Templates** card.
2. Give the template a **Name**, leave **Template published** off until it is ready, and click **Create**. The template opens on the **Template Editor** tab.
3. Write the content in the editor.
4. Add the data the document shows, in the panel on the right:
   - **Add attributes of the instance** picks attributes of the container, such as its name or publication date;
   - **Add related data** builds a widget, a list or a chart of objects linked to the container, as in a [dashboard widget](/opencti/use/dashboards/).
5. Copy the variable name of each widget and paste it where it belongs in the content.
6. Open **Template Preview**, pick a container in **Entity**, and check the result.
7. Click **Save template**, then turn on **Template published** to offer the template to users.

:::

Until you publish it, the template shows **Not published** next to its name and users cannot pick it. A widget that the content does not use shows a warning icon; only those can be deleted. **Import a template** loads a template exported from another platform. The row menu of a template exports or deletes it. The look of generated files, such as colors and logos, comes from its [fintel design](/opencti/configure/fintel-designs/).

:::only{opencti=">=7.260811"}
The default template is already selected when a user generates a file from a container of this type. Turn on **Set as default** in the template form, or choose **Set as default** in its row menu; **Remove default** brings back an empty choice. A type has one default template at most: OpenCTI asks before it replaces the current one.
:::

:::only{opencti=">=7.261030"}
Under **Export defaults**, **Include cover page by default** and **Include back page by default** set the **Page options** that the export dialog starts with for this template. Both are on when you create a template. Turn off the cover page for a document that should open on its content, such as a one-page daily brief. Users can still change both options when they export.
:::

## Rearrange the overview

The **Overview layout customization** card sets the order of the widgets on the **Overview** tab of every object of this type, and which ones take the full width. Drag the widgets into the order you want, turn on **Full width** for the ones that need room, and check the **Preview**. **Reset to default layout**, next to the card title, restores the original layout.

:::only{opencti=">=7.260811"}
## Add custom views

The **Custom Views** tab lists the extra tabs built for this type, made of widgets, that users see on each object. Create one with **Create a new custom view**, import one with **Import a custom view**, or take one from **Import from Hub**. How to build them is in [Add a dashboard tab to entity pages](/opencti/use/dashboards/custom-views/).

The **Status** of each view says whether users see it: **View is enabled** or **View is disabled**. The **Create custom view** panel, and the **Update custom view** panel that **Update** opens, have the **Set as default custom view** switch: the default view replaces the overview as the tab users land on. The row menu of a view also offers **Enable** or **Disable**, and **Set as default** or **Remove default**.
:::

## Check that it works

Open the creation form of the type, for example from :menu[Analyses > Reports] for reports: mandatory attributes show an asterisk, and default values are filled in. A hidden type no longer appears in the menu or in the **Entity type** lists of forms. With **Enforce references** on, change a field of an existing object: the **Reference modification** dialog asks for a reference before it saves. Then check that your connectors still import: their user needs **Bypass mandatory fields** and **Bypass enforced reference** if you made attributes mandatory or references enforced on the types they create.

## What's next

- [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/) to design the statuses of a type.
- [Map CSV files to knowledge](/opencti/configure/mappers/csv/), which use the default values set here.
- [Brand your PDF reports](/opencti/configure/fintel-designs/) for the look of generated documents.
- [Platform parameters](/opencti/configure/parameters/) for the list of hidden types.
