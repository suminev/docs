# Moving a cloud network to a different folder

You can move cloud resources between folders within a single [cloud](../../resource-manager/concepts/resources-hierarchy.md).

{% list tabs %}

- Management console

   To move a [cloud network](../concepts/network.md) to a different folder:

   1. In the [management console]({{ link-console-main }}), go to the folder where the cloud network is located.
   1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
   1. Click ![image](../../_assets/options.svg) next to the network you need and select **{{ ui-key.yacloud.vpc.button_move-vpc-object }}**.
   1. In the window that opens, select the destination folder.
   1. Click **{{ ui-key.yacloud.vpc.button_move-vpc-object }}**.

   {% note info %}

   The cloud network's subnets are also moved to the other folder.

   {% endnote %}

- CLI

   {% include [include](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To move a [cloud network](../concepts/network.md) to a different folder:

   1. View a description of the CLI move cloud network command:

      ```bash
      yc vpc network move --help
      ```

   1. Get the name or ID of the cloud network to move:

      ```bash
      yc vpc network list
      ```
      Result:
      ```text
      +----------------------+-----------+
      |          ID          |   NAME    |
      +----------------------+-----------+
      | encb4ubvmief******** | default   |
      | enc39op1vq9m******** | network-1 |
      | enc772aa2rgq******** | network-2 |
      +----------------------+-----------+
      ```

   1. Get a list of available folders:

      ```bash
      yc resource-manager folder list
      ```

      Result:
      ```text
      +----------------------+------------------------+--------+--------+
      |          ID          |          NAME          | LABELS | STATUS |
      +----------------------+------------------------+--------+--------+
      | b1cs8ie21pk1******** | default                |        | ACTIVE |
      | b1chgf288nvg******** | my-folder-1            |        | ACTIVE |
      | b1cu6g9ielh6******** | my-folder-2            |        | ACTIVE |
      +----------------------+------------------------+--------+--------+
      ```

   1. Move the network by specifying the name or ID of the network and destination folder:

      ```bash
      yc vpc network move <network_name_or_ID> \
        --destination-folder-name <name_of_destination_folder> \
        --destination-folder-id <ID_of_destination_folder>
      ```
      Use either the `--destination-folder-name` parameter or the `--destination-folder-id` parameter.

      If the network is not in the current folder (default folder), specify the source folder using the `--folder-name` or `--folder-id` option.

      Result:

      ```text
      id: enc39op1vq9m********
       folder_id: b1chgf288nvg********
       created_at: "2022-10-06T14:54:48Z"
       name: network-1
       default_security_group_id: enc2ta63h3q2********
      ```

      For more information about the `yc vpc network move` command, see the [CLI reference](../../cli/cli-ref/managed-services/vpc/network/move.md).

- API

   To move a [cloud network](../concepts/network.md) to a different folder, use the [move](../api-ref/Network/move) REST API method for the [Network](../api-ref/Network/index.md) resource or the [NetworkService/Move](../api-ref/grpc/network_service.md#Move) gRPC API call, and provide the following in the request:

   * ID of the cloud network you want to move, in the `networkId` parameter.

      {% include [get-network-id](../../_includes/vpc/get-network-id.md) %}

   * ID of the folder you want to move the cloud network to, in the `destinationFolderId` parameter.

      {% include [get-catalog-id](../../_includes/get-catalog-id.md) %}

{% endlist %}

## Examples {#examples}

### Cloud network in the current folder {#from-default-folder}

Move a cloud network from the current folder by specifying the network name and destination folder name:

{% list tabs %}

- CLI

   ```bash
   yc vpc network move network-1 \
     --destination-folder-name my-folder-1
   ```

{% endlist %}

### Cloud network in a different folder {#from-another-folder}

Move a cloud network from a different folder. Specify the network ID and the source and destination folder IDs:

{% list tabs %}

- CLI

   ```bash
   yc vpc network move enc39op1vq9m******** \
     --folder-id b1chgf288nvg******** \
     --destination-folder-id b1cs8ie21pk1********
   ```

{% endlist %}
