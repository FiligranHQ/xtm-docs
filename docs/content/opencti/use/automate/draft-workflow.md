---
title: Approve drafts step by step
description: Define the statuses a draft goes through before it reaches the OpenCTI knowledge base, who can move it forward, and what happens at each step.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [cti-lead, admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/draft-workflow/
---

The draft workflow makes sure that nothing your analysts prepare in a draft reaches the knowledge base before the right person has reviewed it. It turns draft approval into a review process: a draft moves from status to status through transitions you define, and only a transition that validates the draft sends its content to the knowledge base. This page shows how to design and publish the workflow, restrict who can move a draft, attach actions to each step, and manage drafts with restricted access. If you are new to drafts, read [Drafts](/opencti/use/ingest/drafts/) first.

:::only{opencti="<7.260811"}
The draft workflow arrived in OpenCTI 7.260811. In this version, a draft goes to the knowledge base when a user clicks **Approve draft**. You can still restrict who sees a draft, as described in [Restrict access to drafts](#restrict-access-to-drafts).
:::

::::only{opencti=">=7.260811"}

## How the draft workflow works

Without a workflow, any user who can validate drafts clicks **Approve draft** and the content goes to the knowledge base. Once you publish a workflow, every draft, existing and new, follows it: the draft toolbar shows its status and one button per transition it can take, in place of **Approve draft**.

::diagram{id=opencti-draft-workflow-flow}

- A **status** reuses a status template from :menu[Settings > Taxonomies > Status templates], so names stay the same across entity types.
- A **transition** links two statuses. A draft moves from status A to status B only through a transition from A to B, so nobody skips a step you did not plan.
- **Conditions** on a transition decide who sees and triggers it. **Actions** run when the transition fires, or when a draft enters or leaves a status.
- One workflow applies to all the drafts of the platform.

| Feature | Edition |
| --- | --- |
| Statuses, transitions and the **Validate draft** action | Community and Enterprise |
| Conditions, comments, organization sharing and authorized members actions | Enterprise Edition |

## Before you start

- Your role has the **Manage customization** capability to edit the workflow.
- The status templates you need exist in :menu[Settings > Taxonomies > Status templates], for example `IN_REVIEW`, `APPROVED` and `DECLINED`.
- The users who validate drafts have the **Delete knowledge** capability. A transition with the **Validate draft** action fails for anyone without it, even when its conditions let them see it.

## Design the workflow

:::steps

1. Open :menu[Settings > Customization > Entity types], choose **Draft**, then the **Workflow** tab. An empty workflow invites you to add a first status.
2. Click **Add Status** and choose the status template that every new draft starts in.
3. To add the next step, click the box under a status and choose a template. OpenCTI adds the status and a transition from the previous one. **Add Status** adds a status with no transition yet.
4. To link two existing statuses, drag an arrow from the status where the transition starts to the status where it ends.
5. Click a transition to open its settings. Give it a **Transition name** that users will read on the button, for example `Submit for review` or `Reject`.
6. On the transition that ends the review, turn on **Validate draft** under **Draft validation**. The workflow needs at least one.

   ::screenshot{id=opencti-draft-workflow-editor alt="The Workflow tab of the Draft entity type shows four statuses linked by named transitions, and the Publish button in green."}

:::

Changes save as you go, but drafts keep following the published version until you publish again.

## Restrict and automate transitions

Open a transition to set what it requires and what it does. These settings need an Enterprise Edition license.

**Conditions.** Add filters under **Conditions** so the transition only shows to the right users or drafts: the user, their group or their organization, or attributes of the draft such as its name. A user who can open a draft but does not match the conditions does not see the transition. For example, show `Approve` to the group of senior analysts only.

**Comment.** Turn on **Enable comment** to prompt users for a comment when they trigger the transition, and **Required** to make it mandatory. The comment stays visible on the draft while it remains in the target status, for example to explain a rejection. It holds up to 1,000 characters, and the assignees and participants of the draft who can open it receive it as an alert in the platform, except the user who triggered the transition.

**Organization sharing.** Turn on **Share with organizations** to share every entity and relationship of the draft with organizations when the transition fires. Choose the organizations, or leave the field empty to let the user pick them, optionally, at that moment. **Unshare from organizations** works the same way in reverse. Use it when the people who review the draft belong to another organization and could not otherwise see its content.

**Authorized members.** Turn on **Update authorized members** to change who can access the draft. The members you set replace the existing ones. Besides users, groups and organizations, you can choose dynamic members: the draft creators, its author, its assignees or its participants.

Click a status to run **Update authorized members** when a draft enters it, under **On enter actions**, or leaves it, under **On exit actions**. For example, give the review group access when a draft enters `IN_REVIEW`.

:::example[Review imported indicators before they reach the knowledge base]
Drafts of new Velvet Harbor indicators come from several analysts. Before anything reaches the knowledge base, a member of the **CTI analysts** group must check it. The workflow uses four status templates, with `IN_PROGRESS` as the starting status:

| Transition name | From | To | Who triggers it | Settings |
| --- | --- | --- | --- | --- |
| `Submit for review` | `IN_PROGRESS` | `IN_REVIEW` | The analyst who wrote the draft | No condition |
| `Approve` | `IN_REVIEW` | `APPROVED` | A CTI analyst | **Conditions**: **Is in group** = CTI analysts. **Validate draft** on |
| `Reject` | `IN_REVIEW` | `DECLINED` | A CTI analyst | **Conditions**: **Is in group** = CTI analysts. **Enable comment** and **Required** on |
| `Back to work` | `DECLINED` | `IN_PROGRESS` | The analyst who wrote the draft | No condition |

On the `IN_REVIEW` status, under **On enter actions**, turn on **Update authorized members** and keep **Creators** with the manage level, then add the **CTI analysts** group, so reviewers can open the draft as soon as it is submitted.

A draft can only reach the knowledge base through `Approve`, and only a CTI analyst sees that button. A rejected draft carries the reviewer's comment back to its author, who fixes it and submits it again. The CTI analysts group needs the **Delete knowledge** capability for `Approve` to succeed.
:::

## Publish the workflow

The **Publish** button shows the state of your changes:

| Color | Meaning |
| --- | --- |
| Orange | Changes are not published yet. Click to publish |
| Green | The button reads **Published**: drafts follow this version |
| Red | The workflow has errors. Click to see them |

A workflow can be published when:

- it has exactly one starting status, a status no transition leads to;
- every status can be reached from the starting status through transitions, and every transition has a source and a target;
- two transitions that leave the same status have different names;
- at least one transition has the **Validate draft** action;
- it keeps every status that drafts currently use. To remove such a status, first move those drafts to another status: filter the draft list on it and trigger a transition on each.

To undo your unpublished changes, open the workflow options menu and choose **Restore published version**. To start from an empty canvas, choose **Reset workflow**: this does not publish anything, and drafts keep following the published version.

A status template used in a workflow cannot be deleted. Remove the status from the workflow, publish, then delete the template.

## Move a draft through the workflow

:::steps

1. Open the draft. Its toolbar shows the current **Status**.
2. Click the transition button, or **Next status** when several transitions are available. A transition that runs actions shows how many it requires.
3. Fill what the transition asks for, such as a comment or the organizations to share with, and confirm.

:::

When a transition runs actions in the background, the toolbar shows it as pending with a progress counter. If an action fails, for example a share with an organization, the toolbar reports **Transition failed** with the error. A user with the **Bypass all capabilities** capability clicks **Clear** to release the draft, which can then take a transition again.

:::only{opencti=">=7.261030"}
Users with the **Bypass all capabilities** capability can also move a draft to any status directly from its status in the toolbar, with or without running the actions of the transition.

The status a draft shows follows its position in the workflow: when the two differ, for example after an interrupted transition, OpenCTI corrects the status the next time someone opens the draft. Platform administrators can turn this correction off by setting `workflow:disable_read_repair` (`WORKFLOW__DISABLE_READ_REPAIR`) to `true`; it is `false` by default. See [Technical customization](/reference/opencti/configuration/#technical-customization).
:::

If users are stuck because of a condition, add a transition to another status that only an administrator group can trigger.

::::

## Restrict access to drafts

By default, a draft is visible to every user who can access knowledge. To limit it to some users, click **Authorized members** in the draft toolbar; your role needs the **Manage authorized members** capability.

:::only{opencti=">=7.260811"}
Workflow actions can also set the authorized members of a draft at each step.
:::

Administrators review drafts with restricted access in :menu[Data > Restriction > Restricted drafts], which lists them with their creator, author, assignees, participants and status. To give everyone access back to drafts, for example when their only members left, select them and remove the access restriction from the toolbar. This screen requires the **Bypass all capabilities** capability.

## What's next

- [Drafts](/opencti/use/ingest/drafts/): create, review and approve drafts.
- [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/): statuses for reports, cases and other entities.
- [Status templates](/opencti/configure/taxonomies/): the names your statuses reuse.
