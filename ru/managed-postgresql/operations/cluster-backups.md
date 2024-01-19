---
title: "Управление резервными копиями PostgreSQL"
description: "Вы можете создавать резервные копии и восстанавливать кластеры из имеющихся резервных копий PostgreSQL. Технология Point-in-Time Recovery (PITR) позволяет восстановить состояние кластера на любой момент времени, начиная от создания резервной копии."
---

# Управление резервными копиями в {{ mpg-name }}

Вы можете создавать [резервные копии](../concepts/backup.md) и восстанавливать кластеры из имеющихся резервных копий.

Также {{ mpg-name }} ежедневно создает автоматическую резервную копию. Вы можете [задать время начала резервного копирования](#set-backup-window) и [установить срок хранения](update.md#change-additional-settings) для нее.

## Восстановить кластер из резервной копии {#restore}

Технология Point-in-Time Recovery (PITR) позволяет восстановить состояние кластера на любой момент времени в интервале от создания самой старой полной резервной копии до архивации самого свежего журнала опережающей записи (Write Ahead Log, WAL). Подробнее в разделе [{#T}](../concepts/backup.md).

Восстанавливая кластер из резервной копии, вы создаете новый кластер с данными из резервной копии. Если в каталоге не хватает [ресурсов](../concepts/limits.md) для создания такого кластера, восстановиться из резервной копии не получится. Средняя скорость восстановления из резервной копии — 10 МБайт/с на каждое ядро БД.

Для нового кластера необходимо задать все параметры, обязательные при его создании.

При восстановлении до состояния на текущий момент времени новый кластер будет отражать состояние:

* существующего кластера на момент восстановления;
* удаленного кластера на момент архивации последнего журнала опережающей записи.

{% list tabs group=instructions %}

- Консоль управления {#console}

  {% include [manual-backup-restore](../../_includes/mdb/mpg/note-warn-restore-manual-backup.md) %}

  **Чтобы восстановить из резервной копии существующий кластер:**

  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.postgresql.cluster.switch_backups }}**.
  1. Нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) для нужной резервной копии, затем нажмите **{{ ui-key.yacloud.mdb.cluster.backups.button_restore }}**.
  1. Задайте настройки нового кластера. В списке **{{ ui-key.yacloud.mdb.forms.base_field_folder }}** можно выбрать каталог для нового кластера.
  1. Чтобы восстановить состояние кластера на требуемый момент времени после создания этой резервной копии, задайте нужное значение настройки **{{ ui-key.yacloud.mdb.forms.field_date }}**. Значение можно ввести вручную или выбрать из выпадающего календаря.

     Если оставить настройку без изменений, кластер будет восстановлен в состояние на момент завершения создания резервной копии.

  1. Нажмите кнопку **{{ ui-key.yacloud.mdb.cluster.backups.button_restore }}**.

  **Чтобы восстановить из резервной копии удаленный ранее кластер:**

  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Выберите вкладку **{{ ui-key.yacloud.postgresql.switch_backups }}**.
  1. Найдите нужную резервную копию по времени создания и идентификатору кластера. В колонке **{{ ui-key.yacloud.common.id }}** содержатся идентификаторы в формате `<идентификатор_кластера>:<идентификатор_резервной_копии>`.
  1. Нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) для нужной резервной копии, затем нажмите **{{ ui-key.yacloud.mdb.cluster.backups.button_restore }}**.
  1. Задайте настройки нового кластера. В списке **{{ ui-key.yacloud.mdb.forms.base_field_folder }}** можно выбрать каталог для нового кластера.
  1. Чтобы восстановить состояние кластера на требуемый момент времени после создания этой резервной копии, задайте нужное значение настройки **{{ ui-key.yacloud.mdb.forms.field_date }}**. Значение можно ввести вручную или выбрать из выпадающего календаря.

     Если оставить настройку без изменений, кластер будет восстановлен в состояние на момент завершения создания резервной копии.

  1. Нажмите кнопку **{{ ui-key.yacloud.mdb.cluster.backups.button_restore }}**.
  
  {{ mpg-name }} запустит операцию создания кластера из резервной копии.
  
- CLI {#cli}
  
  {% include [cli-install](../../_includes/cli-install.md) %}
  
  По умолчанию кластер будет восстановлен в тот же каталог, где находится резервная копия. Чтобы восстановить кластер в другом каталоге, укажите идентификатор этого каталога в параметре `--folder-id`.
  
  Чтобы восстановить кластер из резервной копии:
  
  1. Посмотрите описание команды CLI для восстановления кластера {{ PG }}:
  
      ```bash
      {{ yc-mdb-pg }} cluster restore --help
      ```
  
  1. Получите список доступных резервных копий {{ PG }}-кластеров:
  
      ```bash
      {{ yc-mdb-pg }} backup list
      ```
  
      ```text
      +--------------------------+---------------------+----------------------+---------------------+
      |              ID          |      CREATED AT     |  SOURCE CLUSTER ID   |      STARTED AT     |
      +--------------------------+---------------------+----------------------+---------------------+
      | c9qlk4v13uq7********:... | 2020-08-10 12:00:00 | c9qlk4v13uq7******** | 2020-08-10 11:55:17 |
      | ...                                                                                         |
      +--------------------------+---------------------+----------------------+---------------------+
      ```
     
      Время завершения создания резервной копии указано в столбце `CREATED AT` списка доступных резервных копий в формате `yyyy-mm-dd hh:mm:ss` (`2020-08-10 12:00:00` в примере выше). Вы можете восстановить состояние кластера на любой момент времени, начиная от создания резервной копии.
          
  1. Запросите создание кластера из резервной копии:

      
      ```bash
      {{ yc-mdb-pg }} cluster restore \
         --backup-id=<идентификатор_резервной_копии> \
         --time=<время> \
         --name=<имя_кластера> \
         --environment=<окружение> \
         --network-name=<имя_сети> \
         --host zone-id=<зона_доступности>,`
               `subnet-name=<имя_подсети>,`
               `assign-public-ip=<публичный_доступ_к_хосту> \
         --resource-preset=<класс_хоста> \
         --disk-size=<размер_хранилища_ГБ> \
         --disk-type=<тип_диска>
      ```


      Где:

      * `--backup-id` — идентификатор [резервной копии](../concepts/backup.md).
      * `--time` — момент времени, на который нужно восстановить состояние кластера {{ PG }}, в формате `yyyy-mm-ddThh:mm:ssZ`.
      * `--name` — имя кластера.
      * `--environment` — окружение:

          * `PRESTABLE` — для тестирования. Prestable-окружение аналогично Production-окружению и на него также распространяется SLA, но при этом на нем раньше появляются новые функциональные возможности, улучшения и исправления ошибок. В Prestable-окружении вы можете протестировать совместимость новых версий с вашим приложением.
          * `PRODUCTION` — для стабильных версий ваших приложений.

      * `--network-name` — [имя сети](../../vpc/concepts/network.md#network).
      * `--host` — параметры хоста:

          * `zone-id` — [зона доступности](../../overview/concepts/geo-scope.md).

          
          * `subnet-name` — [имя подсети](../../vpc/concepts/network.md#subnet). Необходимо указывать, если в выбранной зоне доступности создано две или больше подсетей.
          * `assign-public-ip` — флаг, который указывается, если требуется [публичный доступ к хосту](../concepts/network.md#public-access-to-a-host): `true` или `false`.


      * `--resource-preset` — [класс хоста](../concepts/instance-types.md#available-flavors).
      * `--disk-size` — размер хранилища в гигабайтах.
      * `--disk-type` — [тип диска](../concepts/storage.md):

          
          * `network-hdd`;
          * `network-ssd`;
          * `local-ssd`;
          * `network-ssd-nonreplicated`.


- {{ TF }} {#tf}

  Используйте {{ TF }} для восстановления:

   * Существующего кластера из резервной копии.
   * Кластера, созданного и удаленного через Консоль управления, CLI или API.

   {% note info %}

   Кластер будет восстановлен в каталог, идентификатор которого указан в [настройках провайдера]({{ tf-provider-link }}) в параметре `folder_id`.

   {% endnote %}

   Для восстановления потребуется идентификатор резервной копии. Получите список доступных резервных копий {{ PG }}-кластеров [с помощью CLI](#list-backups):

   ```bash
   {{ yc-mdb-pg }} backup list
   ```

   ```text
   +--------------------------+---------------------+----------------------+---------------------+
   |            ID            |      CREATED AT     |  SOURCE CLUSTER ID   |      STARTED AT     |
   +--------------------------+---------------------+----------------------+---------------------+
   | c9qlk4v13uq7********:... | 2020-08-10 12:00:00 | c9qlk4v13uq7******** | 2020-08-10 11:55:17 |
   | ...                                                                                         |
   +--------------------------+---------------------+----------------------+---------------------+
   ```

   {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

   **Чтобы восстановить из резервной копии существующий кластер:**

   1. Создайте [конфигурационный файл {{ TF }}](cluster-create.md#create-cluster) для нового кластера.

       При этом не используйте ресурсы баз данных (`yandex_mdb_postgresql_database`) и пользователей (`yandex_mdb_postgresql_user`) — они будут восстановлены из резервной копии.

   1. Добавьте в конфигурационный файл блок `restore`:

       ```hcl
       resource "yandex_mdb_postgresql_cluster" "<имя_кластера>" {
         ...
         restore {
           backup_id = "<идентификатор_резервной_копии>"
           time      = "<время>"
         }
       }
       ```

       Где:

       * `backup_id` — идентификатор резервной копии.
       * `time` — момент времени, на который нужно восстановить состояние {{ PG }}-кластера, начиная со времени создания выбранной резервной копии. Формат: `yyyy-mm-ddThh:mm:ss`.

       {% note info %}

       Если параметр `time` не указан, кластер будет восстановлен на момент завершения создания резервной копии.

       {% endnote %}

   1. Проверьте корректность настроек.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Подтвердите изменение ресурсов.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  {{ TF }} создаст копию существующего кластера. Базы данных и пользователи будут развернуты из выбранной резервной копии.

  **Чтобы восстановить из резервной копии удаленный ранее кластер:**

   1. Создайте [конфигурационный файл {{ TF }}](cluster-create.md#create-cluster) для нового кластера.

       При этом не используйте ресурсы баз данных (`yandex_mdb_postgresql_database`) и пользователей (`yandex_mdb_postgresql_user`) — они будут восстановлены из резервной копии.

   1. Добавьте в этот конфигурационный файл блок `restore` с именем резервной копии, из которой нужно восстановиться:

       ```hcl
       resource "yandex_mdb_postgresql_cluster" "<имя_кластера>" {
         ...
         restore {
             backup_id = "<идентификатор_резервной_копии>"
         }
       }
       ```

       Где `backup-id` — идентификатор резервной копии удаленного кластера.

   1. Проверьте корректность настроек.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Подтвердите изменение ресурсов.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

  {{ TF }} создаст новый кластер. Базы данных и пользователи будут развернуты из резервной копии.

- API {#api}

    Чтобы восстановить кластер из резервной копии, воспользуйтесь методом REST API [restore](../api-ref/Cluster/restore.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/Restore](../api-ref/grpc/cluster_service.md#Restore) и передайте в запросе:

    * Идентификатор требуемой резервной копии в параметре `backupId`. Чтобы узнать идентификатор, [получите список резервных копий в кластере](#list-backups).
    * Момент времени, на который должен быть восстановлен кластер, в параметре `time`.
    * Имя нового кластера, который будет содержать восстановленные из резервной копии данные, в параметре `name`. Имя кластера должно быть уникальным в рамках каталога.

    По умолчанию кластер будет восстановлен в тот же каталог, где находится резервная копия. Чтобы восстановить кластер в другом каталоге, укажите идентификатор этого каталога в параметре `folderId`.

{% endlist %}

## Создать резервную копию {#create-backup}

{% list tabs group=instructions %}

- Консоль управления {#console}
  
  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.postgresql.cluster.switch_backups }}**.
  1. Нажмите кнопку ![image](../../_assets/console-icons/plus.svg) **{{ ui-key.yacloud.mdb.cluster.backups.button_create }}**.

  {% include [no-prompt](../../_includes/mdb/backups/no-prompt.md) %}

- CLI {#cli}
  
  {% include [cli-install](../../_includes/cli-install.md) %}
  
  {% include [default-catalogue](../../_includes/default-catalogue.md) %}
  
  Чтобы создать резервную копию кластера:
  
  1. Посмотрите описание команды CLI для создания резервной копии {{ PG }}:
  
      ```
      {{ yc-mdb-pg }} cluster backup --help
      ```
  1. Запросите создание резервной копии, указав имя или идентификатор кластера:
  
      ```
      {{ yc-mdb-pg }} cluster backup my-pg-cluster
      ```
  
      Имя и идентификатор кластера можно получить со [списком кластеров](cluster-list.md#list-clusters).

- API {#api}

    Чтобы создать резервную копию кластера, воспользуйтесь методом REST API [backup](../api-ref/Cluster/backup.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/Backup](../api-ref/grpc/cluster_service.md#Backup) и передайте в запросе идентификатор кластера в параметре `clusterId`.

    Идентификатор кластера можно получить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

{% endlist %}

{% include [backup-warning](../../_includes/mdb/backups/backup-create-warning.md) %}

## Получить список резервных копий {#list-backups}

{% list tabs group=instructions %}

- Консоль управления {#console}

  Чтобы получить список резервных копий кластера:
  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.postgresql.cluster.switch_backups }}**.

  Чтобы получить список всех резервных копий в каталоге:
  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Выберите вкладку **{{ ui-key.yacloud.postgresql.switch_backups }}**.
  
- CLI {#cli}
  
  {% include [cli-install](../../_includes/cli-install.md) %}
  
  {% include [default-catalogue](../../_includes/default-catalogue.md) %}
  
  Чтобы получить список резервных копий кластеров {{ PG }}, доступных в каталоге по умолчанию, выполните команду:
  
  ```
  {{ yc-mdb-pg }} backup list
  
  +--------------------------+---------------------+----------------------+---------------------+
  |            ID            |      CREATED AT     |  SOURCE CLUSTER ID   |      STARTED AT     |
  +--------------------------+---------------------+----------------------+---------------------+
  | c9qlk4v13uq7********:... | 2020-08-10 12:00:00 | c9qlk4v13uq7******** | 2020-08-10 11:55:17 |
  | c9qpm90p3pcg********:... | 2020-08-09 22:01:04 | c9qpm90p3pcg******** | 2020-08-09 21:30:00 |
  +--------------------------+---------------------+----------------------+---------------------+
  ```

- API {#api}

    Чтобы получить список резервных копий кластера, воспользуйтесь методом REST API [listBackups](../api-ref/Cluster/listBackups.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/ListBackups](../api-ref/grpc/cluster_service.md#ListBackups) и передайте в запросе идентификатор кластера в параметре `clusterId`.

    Чтобы получить список резервных копий всех кластеров {{ mpg-name }} в каталоге, воспользуйтесь методом REST API [list](../api-ref/Backup/list.md) для ресурса [Backup](../api-ref/Backup/index.md) или вызовом gRPC API [BackupService/List](../api-ref/grpc/backup_service.md#List) и передайте в запросе идентификатор каталога в параметре `folderId`.

    Идентификатор кластера можно получить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

{% endlist %}


## Получить информацию о резервной копии {#get-backup}

{% list tabs group=instructions %}

- Консоль управления {#console}

  Чтобы получить информацию о резервной копии существующего кластера:
  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Нажмите на имя нужного кластера и выберите вкладку **{{ ui-key.yacloud.postgresql.cluster.switch_backups }}**.

  Чтобы получить информацию о резервной копии удаленного ранее кластера:
  1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
  1. Выберите вкладку **{{ ui-key.yacloud.postgresql.switch_backups }}**.
  
- CLI {#cli}
  
  {% include [cli-install](../../_includes/cli-install.md) %}
  
  {% include [default-catalogue](../../_includes/default-catalogue.md) %}
  
  Чтобы получить данные о резервной копии кластера {{ PG }}, выполните команду:
  
  ```
  {{ yc-mdb-pg }} backup get <идентификатор_резервной_копии>
  ```
  
  Идентификатор резервной копии можно получить со [списком резервных копий](#list-backups).

- API {#api}

    Чтобы получить данные о резервной копии кластера, воспользуйтесь методом REST API [get](../api-ref/Backup/get.md) для ресурса [Backup](../api-ref/Backup/index.md) или вызовом gRPC API [BackupService/Get](../api-ref/grpc/backup_service.md#Get) и передайте в запросе идентификатор резервной копии в параметре `backupId`.

    Идентификатор резервной копии можно получить со [списком резервных копий](#list-backups).

{% endlist %}

## Задать время начала резервного копирования {#set-backup-window}

{% list tabs group=instructions %}

- Консоль управления {#console}
  
  В консоли управления задать время начала резервного копирования можно при [создании](cluster-create.md) или [изменении кластера](update.md).

- CLI {#cli}

  Чтобы задать время начала резервного копирования, используйте флаг `--backup-window-start`. Время задается в формате `ЧЧ:ММ:СС`.

  ```bash
  {{ yc-mdb-pg }} cluster create \
     --cluster-name <имя_кластера> \
     --environment <окружение> \
     --network-name <имя_сети> \
     --host zone-id=<зона_доступности>,subnet-id=<идентификатор_подсети> \
     --resource-preset <класс_хоста> \
     --user name=<имя_пользователя>,password=<пароль_пользователя> \
     --database name=<имя_БД>,owner=<имя_владельца_БД> \
     --disk-size <размер_хранилища_ГБ>
     --backup-window-start 10:00:00
  ```

  Где `environment` — окружение: `prestable` или `production`.

  Изменить время начала резервного копирования в существующем кластере можно с помощью команды `update`:

  ```bash
  {{ yc-mdb-pg }} cluster update \
     --cluster-name <имя_кластера> \
     --backup-window-start 11:25:00
  ```

- {{ TF }} {#tf}

    1. Откройте актуальный конфигурационный файл {{ TF }} с планом инфраструктуры.

        О том, как создать такой файл, см. в разделе [{#T}](cluster-create.md).

        Полный список доступных для изменения полей конфигурации кластера {{ mpg-name }} см. в [документации провайдера {{ TF }}]({{ tf-provider-mpg }}).

    1. Добавьте к описанию кластера {{ mpg-name }} блок `backup_window_start` в секции `config`:

        ```hcl
        resource "yandex_mdb_postgresql_cluster" "<имя_кластера>" {
          ...
          config {
            ...
            backup_window_start {
              hours   = <час>
              minutes = <минута>
            }
            ...
          }
          ...
        ```

        Где:

        * `hours` — час начала резервного копирования (UTC).
        * `minutes` — минута начала резервного копирования (UTC).

    1. Проверьте корректность настроек.

        {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

    1. Подтвердите изменение ресурсов.

        {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

        {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

- API {#api}

    Чтобы задать время начала резервного копирования, воспользуйтесь методом REST API [update](../api-ref/Cluster/update.md) для ресурса [Cluster](../api-ref/Cluster/index.md) или вызовом gRPC API [ClusterService/Update](../api-ref/grpc/cluster_service.md#Update) и передайте в запросе:

    * Идентификатор кластера в параметре `clusterId`. Его можно получить со [списком кластеров в каталоге](cluster-list.md#list-clusters).
    * Новое время начала резервного копирования в параметре `configSpec.backupWindowStart`.
    * Список полей конфигурации кластера, подлежащих изменению (в данном случае — `configSpec.backupWindowStart`), в параметре `updateMask`.

    {% include [Сброс настроек изменяемого объекта](../../_includes/note-api-updatemask.md) %}

{% endlist %}

## Удалить резервную копию {#delete}

Вы можете удалить только те резервные копии, которые были созданы вручную.

{% list tabs group=instructions %}

- Консоль управления {#console}

    1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
    1. Выберите кластер {{ mpg-name }}, резервную копию которого нужно удалить.
    1. На левой панели выберите раздел **{{ ui-key.yacloud.postgresql.cluster.switch_backups }}**.
    1. Нажмите на значок ![image](../../_assets/console-icons/ellipsis.svg) справа в строке резервной копии, которую вы хотите удалить.
    1. Выберите пункт **{{ ui-key.yacloud.mdb.cluster.backups.button_delete }}**.
    1. Подтвердите удаление и нажмите кнопку **{{ ui-key.yacloud.mdb.cluster.backups.action_delete-backup }}**.

- CLI {#cli}

    {% include [cli-install](../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../_includes/default-catalogue.md) %}

    Чтобы удалить резервную копию:

    1. Посмотрите описание команды CLI для удаления резервной копии {{ PG }}:

        ```bash
        {{ yc-mdb-pg }} backup delete --help
        ```

    1. Запросите удаление резервной копии, указав ее идентификатор:

        ```bash
        {{ yc-mdb-pg }} backup delete <идентификатор_резервной_копии>
        ```

    Идентификатор резервной копии можно получить со [списком резервных копий](#list-backups).

- API {#api}

    Чтобы удалить резервную копию, воспользуйтесь методом REST API [delete](../api-ref/Backup/delete.md) для ресурса [Backup](../api-ref/Backup/index.md) или вызовом gRPC API [BackupService/Delete](../api-ref/grpc/backup_service.md#Delete) и передайте в запросе идентификатор резервной копии в параметре `backupId`.

    Идентификатор резервной копии можно получить со [списком резервных копий](#list-backups).

{% endlist %}
