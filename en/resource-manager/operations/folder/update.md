# Updating a folder

To update a folder name or description, use the management console, CLI, API, or {{ TF }}.

{% list tabs %}

- Management console

   1. Select the folder on the management console [home page]({{ link-console-cloud }}). This page displays folders for the selected cloud. If necessary, [switch to another cloud](../cloud/switch-cloud.md).
   1. Click ![image](../../../_assets/console-icons/ellipsis.svg) next to the folder and select **{{ ui-key.yacloud.common.edit }}**.
   1. Enter a new name and description for the folder.
   1. Click **{{ ui-key.yacloud.iam.cloud.folders.popup-edit_button_save }}**.

- CLI

   {% include [cli-install](../../../_includes/cli-install.md) %}

   1. View the description of the update folder command:

      ```bash
      yc resource-manager folder update --help
      ```
   1. {% include [get-folder-id-or-name](../../../_includes/resource-manager/get-folder-id-or-name.md) %}

   1. Change the folder parameters, e.g., name and description. You can specify the folder to update by its name or ID.

      ```bash
      yc resource-manager folder update default \
        --new-name myfolder \
        --description "this is my default-folder"
      ```

      The command will rename the `default` folder to `myfolder` and update its description. The folder naming requirements are as follows:

      {% include [name-format](../../../_includes/name-format.md) %}

- {{ TF }}

   {% include [terraform-install](../../../_includes/terraform-install.md) %}

   To edit the settings of a folder created with {{ TF }}:

   1. Open the {{ TF }} configuration file and edit the fragment with the folder description.

      {% cut "Example folder description in a {{ TF }} configuration" %}

      ```hcl
      ...
      resource "yandex_resourcemanager_folder" "folder1" {
        cloud_id    = "b1gia12lt1tp********"
        name        = "myfolder"
        description = "this is my default-folder"
      }
      ...
      ```

      {% endcut %}

      For more information about the parameters of the `yandex_resourcemanager_folder` resource in {{ TF }}, see the [provider documentation]({{ tf-provider-resources-link }}/resourcemanager_folder).

   1. Check the configuration using this command:
      ```
      terraform validate
      ```

      If the configuration is correct, you will get this message:

      ```
      Success! The configuration is valid.
      ```

   1. Run this command:
      ```
      terraform plan
      ```

      The terminal will display a list of resources with parameters. No changes will be made at this step. If the configuration contains any errors, {{ TF }} will point them out.

   1. Apply the configuration changes:
      ```
      terraform apply
      ```

   1. Confirm the changes: type `yes` into the terminal and press **Enter**.

      You can check the folder update using the [management console]({{ link-console-main }}) or this [CLI](../../../cli/quickstart.md) command:

      ```
      yc resource-manager folder list-operations <folder_name_or_ID>
      ```

- API

   To update a folder, use the [update](../../api-ref/Folder/update.md) REST API method for the [Folder](../../api-ref/Folder/index.md) resource or the [FolderService/Update](../../api-ref/grpc/folder_service.md#Update) gRPC API call.

{% endlist %}

