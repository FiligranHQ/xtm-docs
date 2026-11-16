---
title: Manage labels, vocabularies and templates
description: Manage the labels, kill chain phases, vocabularies, status templates, case templates and custom fields that classify OpenCTI data, so every team classifies the same way.
sidebar_position: 30
doc_type: how-to
products:
  - opencti
personas: [admin, cti-lead, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/ontologies/
---

Taxonomies are the shared lists that classify OpenCTI data: labels, kill chain phases, the values of drop-down fields, and the templates of statuses and cases. When they are kept tidy, analysts pick the same value for the same thing, and every filter, dashboard and playbook built on it catches everything. This page helps the CTI team lead choose the right list for each need, and shows the administrator where each list lives and how to change it.

## Choose the right list

Decide with the CTI team which list carries what, before analysts start to tag. The wrong choice shows later: a report type kept as a label has no fixed set of values, so spellings multiply and filters miss reports.

| You need | Use | Why |
| --- | --- | --- |
| A free tag analysts put on any object, such as a program name or a review step | **Labels** | Analysts filter, chart and search on labels, and playbooks react to them |
| The values offered in a drop-down field of the data model, such as the types of a report | **Vocabularies** | One controlled list for the field, with aliases that catch other spellings from connectors |
| The stages of an attack model, such as MITRE ATT&CK tactics | **Kill chain phases** | Attack patterns and indicators map to them, and matrix views are drawn from them |
| The steps of a review workflow, such as *New*, *In progress*, *Analyzed* | **Status templates** | Entity workflows are built from them |
| The same tasks for every case of a kind, such as the steps of an incident response | **Case templates** | A case gets the tasks of its template when it is created |

:::only{opencti=">=7.261109"}
When the data model has no field for what your team records, such as the impact of a threat on your members, create a **custom field**: a new typed field, with its own values, added to the entity types you choose. See [Custom fields](#custom-fields).
:::

## Before you start

Each list has its own capability, under **Access to admin functionalities**, so you can delegate one without the others, for example **Manage labels** to the CTI team lead. The **Taxonomies** menu shows when your role has at least one of them.

| List | Capability |
| --- | --- |
| Labels | **Manage labels** |
| Kill chain phases | **Manage kill chain phases** |
| Vocabularies | **Manage vocabularies** |
| Status templates | **Manage status templates** |
| Case templates | **Manage case templates** |

:::only{opencti=">=7.261109"}
Custom fields need **Manage custom fields**, and the Enterprise Edition. Adding a custom field to an entity type needs **Manage customization**.
:::

The [capability reference](/reference/opencti/capabilities/) lists them with the other capabilities of **Access to admin functionalities**.

Write down your classification rules, for example which labels exist and when to use them, before you open the lists to many people.

## Open the taxonomies

Open :menu[Settings > Taxonomies]. The menu on the right leads to **Labels**, **Kill chain phases**, **Vocabularies**, **Status templates** and **Case templates**.

:::only{opencti=">=7.261109"}
With **Manage custom fields**, it also leads to **Custom fields**.
:::

Each list has a create button at the top right and a menu (⋮) on each line to update or delete it.

## Labels

Labels tag any object with a free category, such as a priority, a campaign name or a workflow step. Analysts filter, chart and search on them, and [playbooks](/opencti/use/automate/playbooks/) can react to them: one playbook adds `weekly-digest`, another collects everything with that label.

:::steps

1. In **Labels**, click **Create Label**.
2. Fill the **Value** and pick a **Color**, then click **Create**.

:::

::screenshot{id=opencti-taxonomies-labels alt="The Labels list of the Taxonomies settings, with the Create Label button and labels such as hotel-sector, phishing and scanner, each with its color, next to the menu of the five taxonomies."}

To rename or recolor a label, open its menu (⋮) and choose **Update**.

:::example[Tag the hotel-sector program the same way]
Example CERT runs a sharing program for hotel chains. Analysts used to tag its reports `hotels`, `hospitality` or `hotel sector`, so no filter caught them all.

1. In **Labels**, create the label with **Value** `hotel-sector` and **Color** `#1F7A8C`, the teal of the program. Delete or stop using the other spellings.
2. In **Vocabularies**, open `report_types_ov` and click **Create Vocabulary**: **Name** `member-advisory`, **Description** `Advisory written for the member hotel chains`, **Aliases** `member advisory` and `hotel advisory`.

Every report of the program now carries one label that playbooks, triggers and dashboards filter on, such as the trigger "New hotel-sector reports". **Report types** in the report form offers member-advisory, and a connector that sends the type "hotel advisory" lands on member-advisory through its alias.
:::

## Kill chain phases

Kill chain phases describe the stages of an attack. Attack patterns, indicators and other objects are mapped to them, and the matrix views are drawn from them. They usually arrive with the data, for example with the MITRE ATT&CK connector.

OpenCTI does not impose a model: the **Kill chain name** of a phase decides which model it belongs to. The Lockheed Martin Cyber Kill Chain, the MITRE ATT&CK matrices and the DISARM framework for disinformation can live side by side, each under its own name. Keep the name the connector of a model uses: phases created by hand under another spelling form a separate model, and the matrix views, which show one kill chain at a time, split the techniques between the two.

:::steps

1. In **Kill chain phases**, click **Create Kill Chain phase**.
2. Fill the **Kill chain name**, which groups the phases of one model, such as `mitre-attack`, the **Phase name** and the **Order** that places the phase in the model.
3. Click **Create**.

:::

## Vocabularies

Vocabularies are the values offered in drop-down fields: report types, malware types, case priorities and the other open vocabularies of the STIX standard and of OpenCTI. **Vocabularies** lists the categories, such as `report_types_ov`, with the entity types that use each one. Open a category to see its values, their aliases and how many objects use each.

:::steps

1. Open the category, then click **Create Vocabulary**.
2. Fill the **Name**, then optionally the **Description**, the **Aliases** and the **Order** in the drop-down.
3. Click **Create**.

:::

::screenshot{id=opencti-taxonomies-report-types alt="The values of the report_types_ov vocabulary: the built-in internal-report and threat-report next to member-advisory, with its aliases, its description and the number of reports that use it."}

Rules to know:

- **Aliases** catch data that arrives with another spelling of the value.
- **Hidden**, in the **Update an attribute** panel that the menu (⋮) of a value opens, removes it from the choices of forms without touching the objects that already use it.
- Built-in values cannot be deleted: in their menu, **Delete** gives way to **This item is built-in**. Their names cannot change, nor can the names of the values of a closed category.
- A value used by objects cannot be deleted when a field that uses the category is mandatory: **Delete** gives way to **Some fields in usage are mandatory**.

The [vocabularies reference](/reference/opencti/vocabularies/) lists every category of your version, the entity types and fields that take its values, and the values a new platform starts with.

## Status templates

Status templates are the statuses that [entity workflows](/opencti/use/automate/entity-workflows/) are built from, such as *New*, *In progress* or *Analyzed*. In **Status templates**, click **Create Status Template**, fill the **Name** and the **Color**, then click **Create**. The **Usages** column shows how many workflows use each status. Use the status in the workflow of an entity type in :menu[Settings > Customization > Entity types].

## Case templates

A case template is a list of tasks created with a case, so every incident response follows the same steps.

:::steps

1. In **Case templates**, click **Create Case Template**.
2. Fill the **Name** and the **Description**.
3. In **Tasks**, pick existing task templates, or create one with a **Name** and a **Description**.
4. Click **Create**. Open the template later to add, edit or remove tasks.

:::

On the page of a template, the menu (⋮) of a task offers **Update**, **Unlink** and **Delete**. **Unlink** removes the task from this template only: other templates that share it keep it. **Delete** removes the task template itself, from every case template. Tasks already created in cases are not affected by either.

When analysts create an incident response, a request for information or a request for takedown, they pick templates in **Default case templates** and the case gets their tasks. On an existing case, **Apply a new case template** adds the tasks of more templates. See [Cases](/opencti/use/analyze/cases/).

::::only{opencti=">=7.261109"}
## Custom fields

Custom fields require the Enterprise Edition. A custom field adds to the data model a field your team needs and OpenCTI lacks, such as the impact of a threat on your members or the booking platform an attack targets. You define it once here, then add it to entity types in [Customize entity types](/opencti/configure/entities/#add-custom-fields-to-an-entity-type). Analysts then fill it in the creation and edition forms of those types, read it in the **Entity details** card, and filter, chart and update on it like a built-in field.

Custom fields go on entities, relationships and sightings. Observables cannot hold them.

:::steps

1. In **Custom fields**, click **Create Custom Field**.
2. Choose the **Field type**. It decides the values the field takes, and cannot change later:

   | Field type | Values |
   | --- | --- |
   | **Text** | One line of text |
   | **Markdown** | Formatted text, shown on the full width of the **Details** card |
   | **Integer** | A whole number, optionally between a **Min value** and a **Max value** |
   | **Boolean** | Yes or no |
   | **Date** | A date and time |
   | **Selection list** | One value of the **Select options** you list |
   | **Multiple selection list** | Several values of the **Select options** you list |

3. Fill the **Label**, the name analysts see, unique among custom fields. OpenCTI derives the **Technical name** from it: lowercase letters, numbers and underscores, starting with a letter, prefixed with `x_opencti_cf_`. The technical name names the field in the API, in filters and in STIX exports, and cannot change later.
4. Optionally, add **Aliases**: names of a STIX property, starting with `x_`, that fill the field when an imported object carries them, such as `x_member_impact` from a partner platform.
5. For an **Integer**, set the **Min value** and **Max value** if the values have bounds. For a selection list, type each of the **Select options** and press Enter.
6. Fill the **Description**, which tells analysts what to enter, then click **Create**.

:::

::screenshot{id=opencti-taxonomies-custom-fields alt="The Custom fields list of the Taxonomies settings, with the Member impact selection list and the Affected booking platform text field, their aliases, the Report entity type that uses them, and their technical names."}

The list gives, for each field, its **Aliases**, the entity types it is **Used in**, its **Type** and its **Technical name**. The menu (⋮) of a line offers **Update** and **Delete**:

- **Update** changes the label, the aliases, the description, the bounds and the options. You cannot tighten the bounds of an integer past a value already stored, nor remove an option that an object still uses.
- **Delete** removes the field and, in a background task, its value from every object: OpenCTI asks you to confirm, as the values cannot come back.

Exports carry the values in the OpenCTI extension of each STIX object, as `x_opencti_cf_<name>` properties, and an import fills the field from that property or from one of its aliases. Without the Enterprise Edition, the **Custom fields** screen shows the Enterprise Edition panel instead of the list; the values already stored stay on the objects.

:::example[Record the impact of a threat on the member hotel chains]
Example CERT writes reports for the hotel chains of its sharing program. Its CTI team lead wants each report to say how hard the threat hits the members, so that the weekly dashboard ranks reports by impact.

1. In **Custom fields**, click **Create Custom Field**. **Field type**: **Selection list**. **Label**: `Member impact`, which gives the technical name `x_opencti_cf_member_impact`. **Aliases**: `x_member_impact`, the property a partner CERT sends. **Select options**: `none`, `limited`, `significant`, `severe`. **Description**: `Impact of the threat on the member hotel chains, set by the analyst who writes the report.` Click **Create**.
2. In :menu[Settings > Customization > Entity types], open **Report**, then add **Member impact** in the **Custom fields** card of the **Attributes** tab, with `limited` as its default value.

Every new report now offers **Member impact** in its form, prefilled with `limited`. The analyst who wrote *Harbor Stealer collects card data* set it to `severe`: the **Entity details** card of the report shows it, beside **Affected booking platform**, a text field the team added the same way:

::screenshot{id=opencti-custom-fields-report-values alt="The Entity details card of the report Harbor Stealer collects card data, with its two custom fields filled in: Member impact severe, and Affected booking platform Front desk PMS."}

Analysts filter the reports with `Member impact` = `severe`, and a report from the partner CERT arrives with its impact already set.
:::
::::

## What's next

- [Customize entity types](/opencti/configure/entities/) to make fields mandatory and set default values.
- [Vocabularies reference](/reference/opencti/vocabularies/) for the shipped values.
- [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/) to use your status templates.
