# Удаление каталога

Чтобы удалить каталог, у вас должна быть роль [`editor`](../../../iam/concepts/access-control/roles.md#editor) или выше на этот каталог. Если вы не можете выполнить эту операцию, обратитесь к [администратору](../../../iam/concepts/access-control/roles.md#admin) или [владельцу каталога](../../concepts/resources-hierarchy.md#owner).

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. Выберите каталог на [стартовой странице]({{ link-console-cloud }}) консоли управления. На этой странице отображаются каталоги для выбранного облака. Если необходимо, [переключитесь на другое облако](../cloud/switch-cloud.md).
  
  1. Нажмите значок ![image](../../../_assets/console-icons/ellipsis.svg) напротив каталога и выберите **{{ ui-key.yacloud.common.delete }}**.
  
  1. Укажите срок удаления каталога — период, после которого каталог будет удален. Выберите один из возможных периодов или `{{ ui-key.yacloud.component.iam-delete-folder-or-cloud-dialog.label_delete-now }}`. Срок удаления каталога по умолчанию — 7 дней.

  1. Нажмите **{{ ui-key.yacloud.common.delete }}**.

  Ресурсы будут остановлены, каталог перейдет в статус ожидания удаления `PENDING_DELETION`. Удаление каталога, находящегося в статусе `PENDING_DELETION`, можно отменить. Для этого нажмите на значок ![***](../../../_assets/console-icons/ellipsis.svg) справа от каталога и выберите пункт **{{ ui-key.yacloud.iam.cloud.folders.button_cancel-deletion }}**.

- CLI {#cli}
  
  {% include [cli-install](../../../_includes/cli-install.md) %}

  1. Посмотрите описание команды удаления каталога:
    
      ```bash
      yc resource-manager folder delete --help
      ```

  1. {% include [get-folder-id-or-name](../../../_includes/resource-manager/get-folder-id-or-name.md) %}

  1. Удалите каталог:
    
      ```bash
      yc resource-manager folder delete <идентификатор_каталога>
      ```
      
      Чтобы отобразить информацию о выполняемой операции, не дожидаясь ее завершения, установите флаг `--async`.

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  Чтобы удалить каталог, созданный с помощью {{ TF }}:

  1. Откройте файл конфигурации {{ TF }} и удалите фрагмент с описанием каталога.

     {% cut "Пример описания каталога в конфигурации {{ TF }}" %}

     ```hcl
     ...
     resource "yandex_resourcemanager_folder" "folder1" {
       cloud_id    = "b1gia12......fvsleds"
       name        = "new-folder"
       description = "this is my default-folder"
     }
     ...
     ```

     {% endcut %}

     Более подробную информацию о параметрах ресурса `yandex_resourcemanager_folder` в {{ TF }}, см. в [документации провайдера]({{ tf-provider-resources-link }}/resourcemanager_folder).

  1. В командной строке перейдите в папку, где расположен конфигурационный файл {{ TF }}.

  1. Проверьте конфигурацию командой:
     ```
     terraform validate
     ```
     
     Если конфигурация является корректной, появится сообщение:
     
     ```
     Success! The configuration is valid.
     ```

  1. Выполните команду:
     ```
     terraform plan
     ```
  
     В терминале будет выведен список ресурсов с параметрами. На этом этапе изменения не будут внесены. Если в конфигурации есть ошибки, {{ TF }} на них укажет.

  1. Примените изменения конфигурации:
     ```
     terraform apply
     ```
     
  1. Подтвердите изменения: введите в терминал слово `yes` и нажмите **Enter**.

     Проверить удаление каталога можно в [консоли управления]({{ link-console-main }}) или с помощью команды [CLI](../../../cli/quickstart.md):

     ```
     yc resource-manager folder list
     ```

- API {#api}

  Чтобы удалить каталог, воспользуйтесь методом REST API [delete](../../api-ref/Folder/delete.md) для ресурса [Folder](../../api-ref/Folder/index.md) или вызовом gRPC API [FolderService/Delete](../../api-ref/grpc/folder_service.md#Delete).

{% endlist %}

{% note alert %}

Пока каталог находится в статусе `PENDING_DELETION`, ваши диски, зарезервированные IP-адреса и другие данные продолжают храниться. Стоимость хранения рассчитывается по прежнему тарифу. Следите за тарификацией ресурсов в [сервисе {{ billing-name }}]({{ link-console-billing }}).

{% endnote %}

После завершения периода ожидания каталог переходит в статус `DELETING`. В этом статусе происходит процесс необратимого удаления, занимающий до 72 часов, в результате которого вместе с каталогом будут удалены все его ресурсы.
