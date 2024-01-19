# Migrating data from {{ OS }} to {{ mos-full-name }} using {{ data-transfer-full-name }}

With {{ data-transfer-name }}, you can transfer data from a third-party {{ OS }} source cluster's indexes to {{ mos-name }} indexes. To do this:

1. [Set up the source cluster](#configure-source).
1. [Prepare the test data](#prepare-data).
1. [Configure the target cluster](#configure-target).
1. [Prepare and activate the transfer](#prepare-transfer).
1. [Test the transfer](#verify-transfer).

If you no longer need the resources you created, [delete them](#clear-out).

## Getting started {#before-you-begin}

1. [Enable access to the source {{ OS }} cluster from {{ yandex-cloud }}](../../data-transfer/concepts/network.md#source-external).

1. Create a target {{ mos-name }} cluster:

   {% list tabs %}

   - Manually

      [Create a {{ mos-name }} target cluster](../../managed-opensearch/operations/cluster-create.md) in any suitable configuration with publicly available hosts.

   - Using {{ TF }}

      1. {% include [terraform-install](../../_includes/terraform-install.md) %}
      1. Download the [file with provider settings](https://github.com/yandex-cloud/examples/tree/master/tutorials/terraform/provider.tf). Place it in a separate working directory and specify the parameter values.
      1. Download the [data-transfer-os-mos.tf](https://github.com/yandex-cloud-examples/yc-data-transfer-opensearch-from-onprem-to-cloud/blob/main/data-transfer-os-mos.tf) configuration file to the same working directory.

         This file describes:

         * [Network](../../vpc/concepts/network.md#network).
         * [Subnet](../../vpc/concepts/network.md#subnet).
         * [Security group](../../vpc/concepts/security-groups.md) and rules required to connect to a {{ mos-name }} cluster.
         * {{ mos-name }} target cluster.
         * Transfer.

      1. In the `data-transfer-os-mos.tf` file, specify the variables:

         * `os_admin_password`: {{ mos-name }} admin user password.
         * `transfer_enabled`: Set `0` to ensure that no transfer is created until you [create endpoints manually](#prepare-transfer).

      1. Run the `terraform init` command in the directory with the configuration file. This command initializes the provider specified in the configuration files and enables you to use the provider resources and data sources.
      1. Check that the {{ TF }} configuration files are correct using this command:

         ```bash
         terraform validate
         ```

         If there are any errors in the configuration files, {{ TF }} will point them out.

      1. Create the required infrastructure:

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

         {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

   {% endlist %}

1. Install the utilities:

   * [curl](https://curl.se/) to make requests to clusters.

      ```bash
      sudo apt update && sudo apt install --yes curl
      ```

   * [jq](https://stedolan.github.io/jq/) for JSON file stream processing.

      ```bash
      sudo apt update && sudo apt install --yes jq
      ```

## Set up the source cluster {#configure-source}

Create a user to execute the transfer.

You can provide data from the {{ OS }} cluster as the `admin` user with the `superuser` role; however, it is more secure to create separate users with limited privileges for each job.

1. (Optional) In the source cluster, [create a role]({{ os.docs }}/security-plugin/access-control/users-roles/#create-roles) with the `create_index` and `write` privileges for all indexes (`*`).

1. (Optional) In the source cluster, [create a user]({{ os.docs }}/security-plugin/access-control/users-roles) to execute the transfer and assign the user the role you created.

## Prepare the test data {#prepare-data}

1. In the source cluster, create a test index named `people` and set its schema:

   ```bash
   curl --user <source_cluster_username>:<source_cluster_user_password> \
        --header 'Content-Type: application/json' \
        --request PUT 'https://<{{ OS }}_DATA_host_address>:{{ port-mos }}/people' && \
   curl --user <source_cluster_username>:<source_cluster_user_password> \
        --header 'Content-Type: application/json' \
        --request PUT 'https://<{{ OS }}_DATA_host_address>:{{ port-mos }}/people/_mapping?pretty' -d'
        {
              "properties": {
                 "name": {"type": "text"},
                 "age": {"type": "integer"}
              }
        }
        '
   ```

1. Populate the test index with data:

   ```bash
   curl --user  <source_cluster_username>:<source_cluster_user_password> \
        --header 'Content-Type: application/json' \
        --request POST 'https://<{{ OS }}_DATA_host_address>:{{ port-mos }}/people/_doc/?pretty' -d'
        {
              "name" : "Alice",
              "age" : "30"
        }
        ' && \
   curl --user  <source_cluster_username>:<source_cluster_user_password> \
        --header 'Content-Type: application/json' \
        --request POST 'https://<{{ OS }}_DATA_host_address>:{{ port-mos }}/people/_doc/?pretty' -d'
        {
              "name" : "Robert",
              "age" : "32"
        }
        '
   ```

1. (Optional) Check the data in the test index:

   ```bash
   curl --user <target_cluster_username>:<target_cluster_user_password> \
        --header 'Content-Type: application/json' \
        --request GET 'https://<{{ OS }}_DATA_host_address>:{{ port-mos }}/people/_search?pretty'
   ```

## Configure the target cluster {#configure-target}

1. [Get an SSL certificate](../../managed-opensearch/operations/connect.md#ssl-certificate) to connect to the {{ mos-name }} cluster.

1. (Optional) Create a user to execute the transfer.

   You can provide data to the {{ mos-name }} cluster as the `admin` user with the `superuser` role; however, it is more secure to create separate users with limited privileges for each job.

   1. [Create a role]({{ os.docs }}/security-plugin/access-control/users-roles/#create-roles) with the `create_index` and `write` privileges for all indexes (`*`).

   1. [Create a user](../../managed-opensearch/operations/cluster-users.md) and assign the user this role.

## Prepare and activate the transfer {#prepare-transfer}

1. [Create an endpoint](../../data-transfer/operations/endpoint/index.md#create) for the [source {{ OS }} cluster](../../data-transfer/operations/endpoint/source/opensearch.md#on-premise).

1. [Create an endpoint](../../data-transfer/operations/endpoint/index.md#create) for the [target {{ mos-name }} cluster](../../data-transfer/operations/endpoint/target/opensearch.md).

1. Create a transfer:

   {% list tabs %}

   - Manually

      1. [Create a transfer](../../data-transfer/operations/transfer.md#create) of the **_{{ ui-key.yc-data-transfer.data-transfer.console.form.transfer.console.form.transfer.TransferType.snapshot.title }}_** type that will use the created endpoints.
      1. [Activate the transfer](../../data-transfer/operations/transfer.md#activate) and wait for its status to change to **{{ ui-key.yacloud.data-transfer.label_connector-status-DONE }}**.

   - Using {{ TF }}

      1. In the `data-transfer-os-mos.tf` file, specify the variables:

         * `source_endpoint_id`: ID of the source endpoint.
         * `target_endpoint_id`: ID of the target endpoint.
         * `transfer_enabled`: Set `1` to enable transfer creation.

      1. Check that the {{ TF }} configuration files are correct using this command:

         ```bash
         terraform validate
         ```

         If there are any errors in the configuration files, {{ TF }} will point them out.

      1. Create the required infrastructure:

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      1. [Activate the transfer](../../data-transfer/operations/transfer.md#activate) and wait for its status to change to **{{ ui-key.yacloud.data-transfer.label_connector-status-DONE }}**.

   {% endlist %}

## Test the transfer {#verify-transfer}

Check that the `people` index in the {{ mos-name }} cluster contains the transferred data:

{% list tabs %}

- Bash

   Run this command:

   ```bash
   curl --user <target_cluster_username>:<target_cluster_user_password> \
        --cacert ~/.opensearch/root.crt \
        --header 'Content-Type: application/json' \
        --request GET 'https://<FQDN_of_{{ OS }}_DATA_host>:{{ port-mos }}/people/_search?pretty'
   ```

- {{ OS }} Dashboards

   1. [Connect](../../managed-opensearch/operations/connect.md#dashboards) to the target cluster using {{ OS }} Dashboards.
   1. Select the `Global` tenant.
   1. Open the control panel by clicking ![os-dashboards-sandwich](../../_assets/console-icons/bars.svg).
   1. Under **OpenSearch Dashboards**, select **Discover**.
   1. In the **CHANGE INDEX PATTERN** field, select the `people` index.

{% endlist %}

## Delete the resources you created {#clear-out}

Some resources are not free of charge. To avoid paying for them, delete the resources you no longer need:

1. [Delete the transfer](../../data-transfer/operations/transfer.md#delete).
1. [Delete endpoints](../../data-transfer/operations/endpoint/index.md#delete) for both the source and target.
1. Delete the other resources depending on how they were created:

   {% list tabs %}

   * Manually

      1. [Delete the {{ mos-name }} cluster](../../managed-opensearch/operations/cluster-delete.md).

      1. [Delete the subnet](../../vpc/operations/subnet-delete.md) and [network](../../vpc/operations/network-delete.md).

   * Using {{ TF }}

      1. In the terminal window, switch to the directory containing the infrastructure plan.
      1. Delete the `data-transfer-os-mos.tf` configuration file.
      1. Check that the {{ TF }} configuration files are correct using this command:

         ```bash
         terraform validate
         ```

         If there are any errors in the configuration files, {{ TF }} will point them out.

      1. Confirm updating the resources.

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

         All the resources described in the `data-transfer-os-mos.tf` configuration file will be deleted.

   {% endlist %}
