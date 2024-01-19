# Создать триггер для {{ yds-name }}, который вызывает функцию {{ sf-name }}

 Создайте [триггер для {{ yds-name }}](../../concepts/trigger/data-streams-trigger.md), который будет вызывать [функцию](../../concepts/function.md) {{ sf-name }} при отправке данных в [поток](../../../data-streams/concepts/glossary.md#stream-concepts). 

## Перед началом работы {#before-you-begin}

Для создания триггера вам понадобятся:

* Функция, которую триггер будет вызывать. Если у вас нет функции:

    * [Создайте функцию](../function/function-create.md).
    * [Создайте версию функции](../function/version-manage.md).

* (Опционально) Очередь [Dead Letter Queue](../../concepts/dlq.md), куда будут перенаправляться сообщения, которые не смогла обработать функция. Если у вас нет очереди, [создайте ее](../../../message-queue/operations/message-queue-new-queue.md).

* [Сервисные аккаунты](../../../iam/concepts/users/service-accounts.md) с правами:

    * на вызов функции;
    * на чтение из потока, при отправке данных в который триггер будет запускаться;
    * (опционально) на запись в очередь Dead Letter Queue.

    Вы можете использовать один и тот же сервисный аккаунт или разные. Если у вас нет сервисного аккаунта, [создайте его](../../../iam/operations/sa/create.md).

* Поток, при отправке данных в который триггер будет запускаться.  Если у вас нет потока, [создайте его](../../../data-streams/quickstart/create-stream.md). 

## Создать триггер {#trigger-create}

{% include [trigger-time](../../../_includes/functions/trigger-time.md) %}

{% list tabs group=instructions %}

- Консоль управления {#console}

    1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, в котором хотите создать триггер.

    1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}**.

    1. На панели слева выберите ![image](../../../_assets/console-icons/gear-play.svg) **{{ ui-key.yacloud.serverless-functions.switch_list-triggers }}**.

    1. Нажмите кнопку **{{ ui-key.yacloud.serverless-functions.triggers.list.button_create }}**.

    1. В блоке **{{ ui-key.yacloud.serverless-functions.triggers.form.section_base }}**:

        * Введите имя и описание триггера.
        * В поле **{{ ui-key.yacloud.serverless-functions.triggers.form.field_type }}** выберите **{{ ui-key.yacloud.serverless-functions.triggers.form.label_data-streams }}**.
        * В поле **{{ ui-key.yacloud.serverless-functions.triggers.form.field_invoke }}** выберите **{{ ui-key.yacloud.serverless-functions.triggers.form.label_function }}**.

    1. В блоке **{{ ui-key.yacloud.serverless-functions.triggers.form.section_data-streams }}** выберите поток данных и сервисный аккаунт с правами на чтение из потока данных и запись в него.

    1. (Опционально) В блоке **{{ ui-key.yacloud.serverless-functions.triggers.form.section_batch-settings }}** укажите:

        * размер группы сообщений в байтах. Допустимые значения от 1 Б до 64 КБ, значение по умолчанию — 1 Б.
        * максимальное время ожидания. Допустимые значения от 1 до 60 секунд, значение по умолчанию — 1 секунда.

        Триггер группирует сообщения не дольше указанного времени ожидания и отправляет их в функцию. Суммарный объем данных, которые передаются в функцию, может превышать указанный размер группы, если данные передаются в одном сообщении. Во всех остальных случаях объем данных не превышает размер группы.

    1. В блоке **{{ ui-key.yacloud.serverless-functions.triggers.form.section_function }}** выберите функцию и укажите:

        {% include [function-settings](../../../_includes/functions/function-settings.md) %}

    1. (Опционально) В блоке **{{ ui-key.yacloud.serverless-functions.triggers.form.section_function-retry }}**:

        {% include [repeat-request.md](../../../_includes/functions/repeat-request.md) %}

    1. (Опционально) В блоке **{{ ui-key.yacloud.serverless-functions.triggers.form.section_dlq }}** выберите очередь Dead Letter Queue и сервисный аккаунт с правами на запись в нее.

    1. Нажмите кнопку **{{ ui-key.yacloud.serverless-functions.triggers.form.button_create-trigger }}**.

- CLI {#cli}

    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

    Чтобы создать триггер, который вызывает функцию, выполните команду:

    ```bash
    yc serverless trigger create yds \
      --name <имя триггера> \
      --database <размещение_базы_данных> \
      --stream <имя потока данных> \
      --batch-size 1b \
      --batch-cutoff 1s \
      --stream-service-account-id <идентификатор сервисного аккаунта> \
      --invoke-function-id <идентификатор функции> \
      --invoke-function-service-account-id <идентификатор сервисного аккаунта> \
      --retry-attempts 1 \
      --retry-interval 10s \
      --dlq-queue-id <идентификатор очереди Dead Letter Queue> \
      --dlq-service-account-id <идентификатор сервисного аккаунта>
    ```

    Где:

    * `--name` — имя триггера.
    * `--database` — размещение базы данных {{ ydb-short-name }}, к которой привязан поток {{ yds-name }}.

      Чтобы узнать, где размещена база данных, выполните команду `yc ydb database list`. Размещение базы данных указано в столбце `ENDPOINT`, в параметре `database`, например `/{{ region-id }}/b1gia87mba**********/etn7hehf6g*******`.

    * `--stream` — имя потока данных {{ yds-name }}.
    * `--batch-size` — размер группы сообщений. Необязательный параметр. Допустимые значения от 1 Б до 64 КБ, значение по умолчанию — 1 Б.
    * `--batch-cutoff` — максимальное время ожидания. Необязательный параметр. Допустимые значения от 1 до 60 секунд, значение по умолчанию — 1 секунда. Триггер группирует сообщения не дольше `batch-cutoff` и отправляет их в функцию. Суммарный объем данных, которые передаются в функцию, может превышать `batch-size`, если данные передаются в одном сообщении. Во всех остальных случаях объем данных не превышает `batch-size`.
    * `--stream-service-account-id` — идентификатор сервисного аккаунта, у которого есть права на чтение из потока данных и запись в него.
    
    {% include [trigger-cli-param](../../../_includes/functions/trigger-cli-param.md) %}

    Результат:

    ```text
    id: a1smnfisr5**********
    folder_id: b1gc1t4cb6**********
    created_at: "2022-10-31T10:57:08.234586266Z"
    name: data-streams-trigger
    rule:
      data_stream:
        database: /{{ region-id }}/b1gvlrnlei**********/etn3ege6nj**********
        stream: yds-stream
        service_account_id: aje07l4q4v**********
        batch_settings:
          size: "1"
          cutoff: 1s
        invoke_function:
          function_id: d4e155orh3**********
          function_tag: $latest
          service_account_id: aje07l4q4v**********
          retry_settings:
            retry_attempts: "1"
            interval: 10s
          dead_letter_queue:
            queue_id: yrn:yc:ymq:{{ region-id }}:b1gc1t4cb6**********:queue_dead
            service_account_id: aje07l4q4v**********
    status: ACTIVE
    ```

- API {#api}

  Чтобы создать триггер для {{ yds-name }}, воспользуйтесь методом REST API [create](../../triggers/api-ref/Trigger/create.md) для ресурса [Trigger](../../triggers/api-ref/Trigger/index.md) или вызовом gRPC API [TriggerService/Create](../../triggers/api-ref/grpc/trigger_service.md#Create).

{% endlist %}

## Проверить результат {#check-result}

{% include [check-result](../../../_includes/functions/check-result.md) %}

## См. также {#see-also}

* [Триггер для {{ yds-name }}, который вызывает контейнер {{ serverless-containers-name }}](../../../serverless-containers/operations/data-streams-trigger-create.md).
