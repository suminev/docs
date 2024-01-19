---
title: "Инструкция о том, как вызвать функцию в {{ sf-full-name }}"
description: "Из статьи вы узнаете, как вызвать функцию в {{ sf-full-name }}."
---

# Вызвать функцию

Чтобы любой пользователь мог вызывать функцию без передачи заголовка авторизации, [сделайте ее публичной](./function-public.md). Подробнее о правах читайте в разделе [{#T}](../../security/index.md).

## Вызвать функцию {#invoking-function}

Для примера используется функция, описанная в разделе [{#T}](../function/version-manage.md).

{% list tabs group=instructions %}

- Консоль управления {#console}
    
    1. В [консоли управления]({{ link-console-main }}) перейдите в каталог, в котором находится функция.

    1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}**.

    1. Выберите функцию.

    1. Перейдите на вкладку **{{ ui-key.yacloud.serverless-functions.item.switch_testing }}**.

    1. В поле **{{ ui-key.yacloud.serverless-functions.item.testing.field_tag }}** укажите версию функции, которую хотите вызвать.

    1. В поле **{{ ui-key.yacloud.serverless-functions.item.testing.field_payload-template }}** выберите один из вариантов:

        * **{{ ui-key.yacloud.serverless-functions.item.testing.value_empty }}** — произвольный формат данных.
        * **{{ ui-key.yacloud.serverless-functions.item.testing.value_http-request }}** — формат данных для вызова функции, выступающей в качестве обработчика HTTPS-запросов. Подробнее см. в разделе [Концепции](../../concepts/function-invoke.md).
                * **{{ ui-key.yacloud.serverless-functions.item.testing.value_ymq-trigger }}** — формат данных для функции, которая вызывается триггером для обработки сообщений из очереди.
                * **{{ ui-key.yacloud.serverless-functions.item.testing.value_alice-skill }}** — формат данных для вызова функции, принимающей [запрос](https://yandex.ru/dev/dialogs/alice/doc/request.html) от платформы Яндекс Диалоги.

    1. В поле **{{ ui-key.yacloud.serverless-functions.item.testing.field_payload }}** введите входные данные для тестирования функции.

    1. Нажмите кнопку **{{ ui-key.yacloud.serverless-functions.item.testing.button_run-test }}**.

    1. В разделе **{{ ui-key.yacloud.serverless-functions.item.testing.label_title-test-result }}** в поле **{{ ui-key.yacloud.serverless-functions.item.testing.field_execution-result }}** будет показан статус тестирования. **Важно**: максимальное время выполнения функции до [таймаута](../../operations/function/version-manage.md) (включая начальную инициализацию при первом запуске) — 10 минут.

    1. В поле **{{ ui-key.yacloud.serverless-functions.item.testing.field_function-output }}** появится результат выполнения функции.

- HTTPS {#https}

    Вы можете найти ссылку для вызова функции:
    * в поле **{{ ui-key.yacloud.serverless-functions.item.overview.label_invoke-link }}**. Для этого в [консоли управления]({{ link-console-main }}) выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}** и нажмите на строку с нужной функцией.
    * в параметре `http_invoke_url`. Для этого выполните команду:
        ```
        yc serverless function get <имя функции>
        ```

    {% include [function-list-note](../../../_includes/functions/function-list-note.md) %}

    Для обеспечения безопасности функцию можно вызвать только по протоколу HTTPS. Вызовите ее как обычный HTTPS-запрос, вставив ссылку в адресную строку браузера. Формат ссылки:

    ```
    https://{{ sf-url }}/<ID функции>
    ```
    
    Вы можете вызвать определенную версию функции с помощью параметра `tag`. По умолчанию вызывается функция с тегом `$latest`.

    * Пример вызова функции без дополнительных параметров:

        ```
        https://{{ sf-url }}/b09bhaokchn9pnbrlseb
        ```

        На странице появится ответ:

        ```
        Hello, World!
        ```

    * Пример вызова функции с добавлением в URL параметра `name`:

        ```
        https://{{ sf-url }}/b09bhaokchn9pnbrlseb?name=<имя пользователя>
        ```

        На странице появится ответ:

        ```
        Hello, Username!
        ```
    * Пример вызова определенной версии функции с добавлением в URL параметра `tag`:
      
        ```
        https://{{ sf-url }}/b09bhaokchn9pnbrlseb?tag=<тег версии>
        ```
            
- CLI {#cli}

    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

    Вы можете вызвать определенную версию функции с помощью параметра `--tag`. По умолчанию вызывается функция с тегом `$latest`.

    {% include [function-list-note](../../../_includes/functions/function-list-note.md) %}

    * Вызовите функцию, указав в параметре имя для приветствия:

        ```
        yc serverless function invoke <идентификатор функции> -d '{"queryStringParameters": {"name": "Username"}}'
        ```

        Результат:

        ```    
        {"statusCode": 200, "headers": {"Content-Type": "text/plain"}, "isBase64Encoded": false, "body": "Hello, Username!"}
        ```
    * Вызовите определенную версию функции с помощью параметра `--tag`:
    
        ```
        yc serverless function invoke <идентификатор функции> --tag <тег версии функции>
        ```


- {{ yandex-cloud }} Toolkit {#yc-toolkit}

    Вызвать функцию можно с помощью [плагина {{ yandex-cloud }} Toolkit](https://github.com/yandex-cloud/ide-plugin-jetbrains) для семейства IDE на [платформе IntelliJ](https://www.jetbrains.com/ru-ru/opensource/idea/) от [JetBrains](https://www.jetbrains.com/).


{% endlist %}

Подробнее о том, какая должна быть структура функции для вызова разными способами (HTTPS, CLI), читайте в разделе [{#T}](../../concepts/function-invoke.md).