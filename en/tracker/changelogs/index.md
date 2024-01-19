---
title: "{{ tracker-full-name }} revision history for December 2023"
description: "See below the {{ tracker-full-name }} revision history for December 2023."
---

# {{ tracker-full-name }} revision history for December 2023

* [Updates](#top-news)
* [Fixes and improvements](#fixes)

## Updates {#top-news}

### "Status" button in project and portfolio comments {#set-status-via-comment}

We have added the ![](../../_assets/console-icons/heart-pulse.svg) **Status** button you can use to change a status when publishing comments to projects and portfolios.

### Employee's position in queue access rights settings {#show-position}

When setting up [queue access rights](../manager/queue-access.md) on the selected employee's page and in the **Issues with component** section, you can now see employees' positions. Positions are based on data from {{ ya-360 }}. Positions not specified in the {{ ya-360 }} admin panel will not be shown in {{ tracker-name }}.

### Copying and deleting transition settings in the workflow editor {#settings-workflow-editor}

The workflow editor now allows you to copy and delete individual transition settings. Open the transition and go to **Automations**, **Transition conditions**, or **Transition screens**. In these settings, click ![](../../_assets/console-icons/ellipsis.svg) to access the following actions:

* Copy the setting to another transition
* Paste the setting from another transition
* Clear the setting contents

Changes will only affect the transition in which the relevant action is selected.

### Integrations in queue settings are out of beta {#queue-integration}

Earlier, we [announced](2308.md#queue-integrations) a beta version of the integrations page in queue settings. Now, the new integrations page is out of beta and available to all {{ tracker-name }} users. It covers all integration settings: with email, {{ forms-full-name }}, and other services.

## Fixes and improvements {#fixes}

### New interface for the organization fields list page {#new-fields-page}

The [**Administration** → **Issue fields**]({{ link-admin-fields }}) page is now available in the new interface. The page consolidates all [issue fields](../user/create-param.md) of your organization, along with their types, keys, and descriptions. Fields are grouped by categories. In addition to viewing existing fields, the page supports search by field name or key, as well as the creation of new fields.

Non-administrative organization users can view the issue field list via a direct [{{ link-admin-fields }}]({{ link-admin-fields }}) link.
