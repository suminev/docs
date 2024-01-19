---
title: "How to create a {{ GP }} connection in {{ datalens-full-name }}"
description: "In this tutorial, you will learn how to connect to {{ GP }} in {{ datalens-full-name }}."
---

# Creating a {{ GP }} connection

{% include [connection-note](../../../_includes/datalens/datalens-connection-note.md) %}

## Connecting to {{ GP }} {#greenplum-connection}

To create a {{ GP }} connection:


{% include [datalens-workbooks-collections-note](../../../_includes/datalens/operations/datalens-workbooks-collections-note.md) %}



1. Go to the [connections page]({{ link-datalens-main }}/connections).


1. Click **Create connection**.
1. Select a **Greenplum** connection.


1. Select the connection type:

   {% list tabs %}

   - Select in a folder

      {% include [datalens-db-select](../../../_includes/datalens/datalens-db-select.md) %}

      * **Cluster**: Specify a cluster from the list of available {{ GP }} clusters. Cluster settings must have the **{{ datalens-short-name }} access** option enabled. If you do not have an available cluster, click **Create new**.

         {% include [datalens-cluster-list](../../../_includes/datalens/datalens-cluster-list.md) %}

      * **Hostname**: Select the host name from the list of hosts available in the {{ GP }} cluster. You can select multiple hosts. If you are unable to connect to the first host, {{ datalens-short-name }} will select the next one from the list.
      * **Port**: Specify the {{ GP }} connection port. The default port is {{ port-mgp }}.
      * **Path to database**: Specify the name of the database to connect to.
      * **Username**: Specify the username for the {{ GP }} connection.
      * **Password**: Enter the password for the user.
      * **Cache TTL in seconds**: Specify the cache time-to-live or leave the default value. The recommended value is 300 seconds (5 minutes).
      * **Raw SQL level**: Enables you to use an ad-hoc SQL query to [generate a dataset](../../concepts/dataset/settings.md#sql-request-in-datatset).

   - Specify manually

      {% include [datalens-db-specify-mannualy](../../../_includes/datalens/datalens-db-specify-mannualy.md) %}

      * **Hostname**: Specify the path to a master host or a {{ GP }} master host IP address. You can specify multiple hosts in a comma-separated list. If you are unable to connect to the first host, {{ datalens-short-name }} will select the next one from the list.
      * **Port**: Specify the {{ GP }} connection port. The default port is 5432.
      * **Path to database**: Specify the name of the database to connect to.
      * **Username**: Specify the username for the {{ GP }} connection.
      * **Password**: Enter the password for the user.
      * **Cache TTL in seconds**: Specify the cache time-to-live or leave the default value. The recommended value is 300 seconds (5 minutes).
      * **Raw SQL level**: Enables you to use an ad-hoc SQL query to [generate a dataset](../../concepts/dataset/settings.md#sql-request-in-datatset).

   {% endlist %}



1. Click **Create connection**.
1. Enter a name for the connection and click **Create**.

{% include [datalens-check-host](../../../_includes/datalens/operations/datalens-check-host.md) %}

## Additional settings {#additional-settings}

{% include [datalens-db-connection-export-settings](../../../_includes/datalens/operations/datalens-db-connection-export-settings.md) %}
