# Управление группами шардов в кластере {{ CH }}

Вы можете сгруппировать несколько [шардов](../concepts/sharding.md) кластера {{ CH }} в _группу шардов_ и затем размещать таблицы в этой группе.

## Получить список групп шардов в кластере {#list-shard-groups}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-clickhouse }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.clickhouse.cluster.switch_shard-groups }}**.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы получить список групп шардов в кластере, выполните команду:

  ```bash
  {{ yc-mdb-ch }} shard-groups list \
     --cluster-name=<имя_кластера>
  ```

  Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

- API {#api}

  Чтобы получить список групп шардов в кластере, воспользуйтесь методом REST API [listShardGroups](../api-ref/Cluster/listShardGroups.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/ListShardGroups](../api-ref/grpc/cluster_service.md#ListShardGroups) и передайте в запросе идентификатор требуемого кластера в параметре `clusterId`.

  Чтобы узнать идентификатор, [получите список кластеров в каталоге](cluster-list.md#list-clusters).

{% endlist %}

## Получить детальную информацию о группе шардов {#get-shard-group}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-clickhouse }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.clickhouse.cluster.switch_shard-groups }}**.
  1. Выберите группу шардов для просмотра детальной информации о ней.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы получить детальную информацию о группе шардов в кластере, выполните команду:

  ```bash
  {{ yc-mdb-ch }} shard-groups get \
    --cluster-name=<имя_кластера> \
    --name=<имя_группы_шардов>
  ```

  Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

- API {#api}

  Чтобы получить детальную информацию о группе шардов, воспользуйтесь методом REST API [getShardGroup](../api-ref/Cluster/getShardGroup.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/GetShardGroup](../api-ref/grpc/cluster_service.md#GetShardGroup) и передайте в запросе:
  * Идентификатор требуемого кластера в параметре `clusterId`. Чтобы узнать идентификатор, [получите список кластеров в каталоге](cluster-list.md#list-clusters).
  * Имя группы шардов в параметре `shardGroupName`. Чтобы узнать имя, [получите список групп шардов](#list-shard-groups) в кластере.

{% endlist %}

## Создать группу шардов {#create-shard-group}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-clickhouse }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.clickhouse.cluster.switch_shard-groups }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.mdb.shard-groups.button_add-group }}**.
  1. Заполните поля формы и нажмите кнопку **{{ ui-key.yacloud.common.apply }}**.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы создать группу шардов в кластере, выполните команду:

  ```bash
  {{ yc-mdb-ch }} shard-groups create \
    --cluster-name=<имя_кластера> \
    --name=<имя_группы_шардов> \
    --description=<описание_группы_шардов> \
    --shards=<список_имен_шардов>
  ```

  Где `--shards` — список имен шардов, которые нужно включить в группу.

  Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

  Имена шардов можно запросить со [списком шардов в кластере](shards.md#list-shards).

- {{ TF }} {#tf}

  1. Откройте актуальный конфигурационный файл {{ TF }} с планом инфраструктуры.

     О том, как создать такой файл, см. в разделе [Создание кластера](cluster-create.md).
  1. Добавьте к описанию кластера {{ mch-name }} блок описания группы шардов `shard_group`.

     ```hcl
     resource "yandex_mdb_clickhouse_cluster" "<имя_кластера>" {
       ...
       shard_group {
         name        = "<имя_группы_шардов>"
         description = "<необязательное_описание_группы_шардов>"
         shard_names = [
           # Список шардов, входящих в группу
           "<имя_шарда_1>",
           ...
           "<имя_шарда_N>"
         ]
       }
     }
     ```

  1. Проверьте корректность настроек.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Подтвердите изменение ресурсов.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-resources-link }}/mdb_clickhouse_cluster).

  {% include [Terraform timeouts](../../_includes/mdb/mch/terraform/timeouts.md) %}

- API {#api}

  Чтобы создать группу шардов, воспользуйтесь методом REST API [createShardGroup](../api-ref/Cluster/createShardGroup.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/CreateShardGroup](../api-ref/grpc/cluster_service.md#CreateShardGroup) и передайте в запросе:
  * Идентификатор кластера, в котором требуется создать группу, в параметре `clusterId`. Чтобы узнать идентификатор, [получите список кластеров в каталоге](cluster-list.md#list-clusters).
  * Имя группы шардов в параметре `shardGroupName`.
  * Список имен шардов, которые требуется включить в группу, в параметре `shardNames`. Чтобы узнать имена, [получите список шардов](shards.md#list-shards) в кластере.
  * При необходимости, описание группы шардов в параметре `description`.

{% endlist %}

## Изменить группу шардов {#update-shard-group}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-clickhouse }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.clickhouse.cluster.switch_shard-groups }}**.
  1. Нажмите на значок ![image](../../_assets/console-icons/ellipsis.svg) для нужной группы шардов и выберите пункт **{{ ui-key.yacloud.common.edit }}**.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы изменить группу шардов в кластере, выполните команду:

  ```bash
  {{ yc-mdb-ch }} shard-groups update \
    --cluster-name=<имя_кластера> \
    --name=<имя_группы_шардов> \
    --description=<новое_описание_группы_шардов> \
    --shards=<новый_список_имен_шардов>
  ```

  Где `--shards` — новый список имен шардов, которые нужно включить в группу.

  Эта команда заменяет существующий список шардов в группе новым, который был передан команде в параметре `--shards`. Перед выполнением команды убедитесь, что вы включили в новый список все необходимые шарды.

  Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

  Имя группы шардов можно запросить со [списком групп шардов в кластере](#list-shard-groups).

  Имена шардов можно запросить со [списком шардов в кластере](shards.md#list-shards).

- {{ TF }} {#tf}

  1. Откройте актуальный конфигурационный файл {{ TF }} с планом инфраструктуры.

     О том, как создать такой файл, см. в разделе [Создание кластера](cluster-create.md).
  1. Измените в описании кластера {{ mch-name }} блок `shard_group` с нужной группой шардов:

     ```hcl
     resource "yandex_mdb_clickhouse_cluster" "<имя_кластера>" {
       ...
       shard_group {
         name        = "<новое_имя_группы_шардов>"
         description = "<новое_описание_группы_шардов>"
         shard_names = [
           # Новый список входящих в группу шардов
           "<имя_шарда_1>",
           ...
           "<имя_шарда_N>"
         ]
       }
     }
     ```

  1. Проверьте корректность настроек.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Подтвердите изменение ресурсов.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-resources-link }}/mdb_clickhouse_cluster).

  {% include [Terraform timeouts](../../_includes/mdb/mch/terraform/timeouts.md) %}

- API {#api}

  Чтобы изменить группу шардов, воспользуйтесь методом REST API [updateShardGroup](../api-ref/Cluster/updateShardGroup.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/UpdateShardGroup](../api-ref/grpc/cluster_service.md#UpdateShardGroup) и передайте в запросе:
  * Идентификатор кластера, в котором требуется изменить группу, в параметре `clusterId`. Чтобы узнать идентификатор, [получите список кластеров в каталоге](cluster-list.md#list-clusters).
  * Имя группы шардов в параметре `shardGroupName`. Чтобы узнать имя, [получите список групп шардов](#list-shard-groups) в кластере.
  * При необходимости, новое описание группы шардов в параметре `description`.
  * При необходимости, новый список имен шардов, которые требуется включить в группу, в параметре `shardNames`. Чтобы узнать имена, [получите список шардов](shards.md#list-shards) в кластере. Этот список заменит собой текущий: убедитесь, что вы включили в новый список все необходимые шарды.
  * Имена изменяемых параметров в параметре `updateMask`.

  {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}

{% endlist %}

## Удалить группу шардов {#delete-shard-group}

Удаление группы шардов не затрагивает входящие в нее шарды — они остаются в кластере.

Таблицы, созданные поверх удаляемой группы, остаются, но становятся неработоспособными: попытки выполнить запрос к ним приведут к ошибкам. Однако такие таблицы можно удалить до или после удаления группы шардов.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-clickhouse }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.clickhouse.cluster.switch_shard-groups }}**.
  1. Нажмите на значок ![image](../../_assets/console-icons/ellipsis.svg) для нужной группы шардов и выберите пункт **{{ ui-key.yacloud.common.delete }}**.

- CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  Чтобы удалить группу шардов в кластере, выполните команду:

  ```bash
  {{ yc-mdb-ch }} shard-groups delete \
     --cluster-name=<имя_кластера> \
     --name=<имя_группы_шардов>
  ```

  Имя кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

  Имя группы шардов можно запросить со [списком групп шардов в кластере](#list-shard-groups).

- {{ TF }} {#tf}

  1. Откройте актуальный конфигурационный файл {{ TF }} с планом инфраструктуры.

     О том, как создать такой файл, см. в разделе [Создание кластера](cluster-create.md).
  1. Удалите из описания кластера {{ mch-name }} блок описания нужной группы шардов `shard_group`.
  1. Проверьте корректность настроек.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Введите слово `yes` и нажмите **Enter**.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  Подробнее см. в [документации провайдера {{ TF }}]({{ tf-provider-resources-link }}/mdb_clickhouse_cluster).

  {% include [Terraform timeouts](../../_includes/mdb/mch/terraform/timeouts.md) %}

- API {#api}

  Чтобы удалить группу шардов, воспользуйтесь методом REST API [deleteShardGroup](../api-ref/Cluster/deleteShardGroup.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/DeleteShardGroup](../api-ref/grpc/cluster_service.md#DeleteShardGroup) и передайте в запросе:
  * Идентификатор кластера, из которого требуется удалить группу, в параметре `clusterId`. Чтобы узнать идентификатор, [получите список кластеров в каталоге](cluster-list.md#list-clusters).
  * Имя группы шардов в параметре `shardGroupName`. Чтобы узнать имя, [получите список групп шардов](#list-shard-groups) в кластере.

{% endlist %}

{% include [clickhouse-disclaimer](../../_includes/clickhouse-disclaimer.md) %}