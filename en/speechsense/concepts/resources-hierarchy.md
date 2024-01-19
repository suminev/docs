# {{ speechsense-name }} resource hierarchy

To manage dialogs in {{ speechsense-name }}, you can use spaces, connections, and projects. 

Each project and connection belongs to a single space. There are no projects and connections outside of a space.

Spaces are owned by organizations, and organizations do not interact with one another. The resources of one organization cannot interact with the resources of another one. Organizations are managed by [{{ org-full-name }}](../../organization/index.yaml).

You can [set up access](#access-rights-inheritance) in {{ speechsense-name }} at the space or project level. By default, a new organization member has no access to resources within the organization. Permissions must be granted to new members explicitly.

## {{ speechsense-name }} resources {#resources}

### Space {#space}

A _space_ is an isolated {{ speechsense-name }} entity within which you can create connections and projects. In the {{ yandex-cloud }} [resource hierarchy](../../overview/roles-and-resources.md), {{ speechsense-name }} spaces are at the same level with [{{ ml-platform-name }} communities](../../datasphere/concepts/community.md) and [clouds](../../resource-manager/concepts/resources-hierarchy.md#cloud).

Each space has a billing account linked to it. You can use one account to [pay for resources](../pricing.md) in multiple spaces or link a separate account to each space.

### Connection {#connection}

A _connection_ is a {{ speechsense-name }} entity that contains:

* [Uploaded conversation records](../operations/data/upload-data.md).
* Record types: audio or text messages.
* Record metadata received from PBX, CRM systems, or other sources.

   Metadata includes parameters such as names and IDs of the operator and client, conversation date, and conversation language. You can use metadata to generate [reports](reports.md) for analyzing a [dialog](dialogs.md) or a group of dialogs.

If groups of dialogs have different metadata sets or record types, you can create a separate connection for each group.

### Project {#project}

A _project_ is an isolated {{ speechsense-name }} entity within which you can manage dialogs. For example, in a project, you can [view a dialog](../operations/data/manage-dialogs.md#view-dialog) or [build a report](../operations/data/manage-reports.md#build-a-report). In the {{ yandex-cloud }} resource hierarchy, {{ speechsense-name }} projects are on the same level as [{{ ml-platform-name }} projects](../../datasphere/concepts/project) and [folders](../../resource-manager/concepts/resources-hierarchy.md#folder).

A project can be viewed as a slice of data uploaded to a connection. When [creating a project](../operations/project/create.md), you select a connection and [set up filters for dialogs](dialogs.md#filters). As a result, filtered dialogs from the connection are added to the project.

Use multiple projects to group dialogs. For example, you can designate a separate project for each client or product of your company.

## Inheritance of access rights {#access-rights-inheritance}

When a user performs an operation in {{ speechsense-name }}, [{{ iam-full-name }}](../../iam/index.yaml) checks the user's access rights. These rights are inherited:

* Rights to an organization extend to the organization's resources, including spaces, connections, and projects.
* Rights to a space extend to all connections and projects within that space.
* Rights to a project extend to all dialogs and reports within that project.

No access rights are granted to connections. Connections are subject to space rights. To learn more about the roles available in the service, see [{#T}](../security/index.md).

> **Examples**:
>
> * A user with the `speech-sense.spaces.creator` role at the organization level can create a space.
> * A user with the `speech-sense.viewer` (`{{ roles-speechsense-viewer }}` in {{ speechsense-name }}) role at the space level can view data for a space, as well as its nested connections and projects.
> * A user with the `speech-sense.admin` (`{{ roles-speechsense-admin }}` in {{ speechsense-name }}) at the project level can perform any action within this project alone.

A user will not be able to access a space without space or organization level rights.
