---
title: Protect sensitive configuration
description: Limit who can change the OpenCTI settings that can break ingestion or hide data, such as built-in roles and groups, TLP markings and the platform organization.
sidebar_position: 8
doc_type: how-to
products:
  - opencti
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/protect-sensitive-configuration/
---

Some OpenCTI settings can stop ingestion, hide data from users or break automations when they change. Sensitive configuration protection, on by default, lets only a few administrators change them. This page shows what is protected, how to give administrators the right to change it, and how to change the protected scope.

## What is protected

A protected setting shows in red with a **Danger Zone** chip. Users without the right still see it, but its actions are disabled for them.

| Area | What cannot change without the right | Configuration key under `protected_sensitive_config` |
| --- | --- | --- |
| Roles | The roles named in `protected_names`: Administrator, Connector and Default | `roles` |
| Groups | The groups named in `protected_names`: Administrators, Connectors and Default | `groups` |
| Marking definitions | The markings listed in `protected_definitions`: every TLP and PAP marking | `markings` |
| Rules | Turning inference rules on or off | `rules` |
| Platform organization | Setting or changing the **Platform main organization** in [Set platform security policies](/opencti/administer/policies/) | `platform_organization` |
| Enterprise Edition | Turning Enterprise Edition on or off | `ce_ee_toggle` |
| File indexing | Starting, pausing or resetting file indexing | `file_indexing` |
| Connectors | Resetting the state of a connector | `connector_reset` |

::screenshot{id=opencti-sensitive-configuration-roles alt="The Roles list, where the Administrator, Connector and Default roles carry a Danger Zone chip and the Analyst role does not."}

## Who has the right

**Allow modification of sensitive configuration** is a setting of a role, at the top of its **Capabilities** tab. A user has the right when one of their roles has this setting on.

- Out of the box, no role has it, not even Administrator: **Bypass all capabilities** does not include it.
- The platform administrator, the account defined by `app:admin:email`, always has it.
- The setting itself is not protected: any user with **Manage credentials** can turn it on in a role that is not protected. Give **Manage credentials** only to trusted administrators.

Treat this protection as a guard against mistakes in the interface, not as a separate layer of access control. What a user may do still comes from the capabilities of their roles: see [Grant rights with roles](/opencti/administer/roles/).

## Before you start

- Your role has the **Manage credentials** capability, to create roles and groups.
- To change a protected role or group afterwards, such as Administrators, you are logged in as the platform administrator or already have the right.

## Let administrators change protected settings

Give the right through a dedicated role and group, so you can see at a glance who holds it.

:::steps

1. Open :menu[Settings > Security > Roles] and click **Create Role**. Name it, for example, Danger Zone Administration, and click **Create**.
2. Open the role, click **Update**, and check **Allow modification of sensitive configuration** at the top of the **Capabilities** tab.
3. Open :menu[Settings > Security > Groups] and click **Create Group**. Name it, for example, Danger Zone Administrators.
4. Open the group, click **Update**, check the role in its **Roles** tab, and add the people who may change protected settings in its **Members** tab.

:::

::::example[Let two platform engineers change protected settings]
At Example CERT, only the two engineers who run the platform may change the built-in groups and the TLP markings.

- Role **Name**: `Danger Zone Administration`, with **Allow modification of sensitive configuration** checked and no other capability.
- Group **Name**: `Danger Zone Administrators`, **Description**: `Platform engineers allowed to change protected settings`, with the Danger Zone Administration role.
- **Members**: the two engineers, who also stay in Administrators for their other rights.

The engineers can now update the Connectors group and the TLP:AMBER marking. Other administrators still see these settings in red, with the **Danger Zone** chip and their actions disabled.
::::

To give the right to all your administrators instead, add the Danger Zone Administration role to the Administrators group. That group is itself protected, so do it as the platform administrator, or as a member of the new group.

## Check that it works

Log in as a member of the new group and open :menu[Settings > Security > Roles], then the Administrator role: its **Update** button is enabled. For a user without the right, the role shows the **Danger Zone** chip and **Update** is disabled.

## Change what is protected

The protected scope is set in the configuration, under `protected_sensitive_config`. Each area has an `enabled` key, and roles, groups and markings take the list of protected names. A list replaces the default one: repeat the built-in names you want to keep.

:::example[Protect the CTI team and let analysts manage inference rules]
Example CERT wants the CTI analysts group and its Analyst role to be as hard to change as the built-in ones, and lets its senior analysts turn inference rules on and off. In the docker compose file, the `opencti` service gets:

```yaml
services:
  opencti:
    environment:
      PROTECTED_SENSITIVE_CONFIG__GROUPS__PROTECTED_NAMES: '["Administrators","Connectors","Default","CTI analysts"]'
      PROTECTED_SENSITIVE_CONFIG__ROLES__PROTECTED_NAMES: '["Administrator","Connector","Default","Analyst"]'
      PROTECTED_SENSITIVE_CONFIG__RULES__ENABLED: 'false'
```

After a restart, the CTI analysts group and the Analyst role show the **Danger Zone** chip, and turning an inference rule on or off no longer needs **Allow modification of sensitive configuration**.
:::

To turn the whole protection off, set `PROTECTED_SENSITIVE_CONFIG__ENABLED=false` and restart. Every key and its default is in [Protect Sensitive Configurations](/reference/opencti/configuration/#protect-sensitive-configurations).

## What's next

- [Grant rights with roles](/opencti/administer/roles/), for the capabilities a role can carry.
- [Organize users in groups](/opencti/administer/groups/), to manage the members of the new group.
- [Set platform security policies](/opencti/administer/policies/), where the platform organization is set.
