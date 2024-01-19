---
title: "Удалить метку функции"
description: "Следуя данной инструкции, вы сможете удалить метку функции."
---

# Удалить метку функции

{% list tabs group=instructions %}

- CLI {#cli}
  
    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}
    
    Чтобы удалить метку функции, выполните команду:

    ```
    yc serverless function remove-labels <имя функции> --labels <ключ>
    ```

    Результат:

    ```
    id: b097d9ous3ge********
    folder_id: aoek49ghmknn********
    created_at: "2019-06-06T11:36:48.688Z"
    name: my-beta-function
    description: My beta function.
    labels:
      version: beta
    log_group_id: eolv6578frac********
    http_invoke_url: https://{{ sf-url }}/b097d9ous3ge********
    status: ACTIVE
    ```

- {{ TF }} {#tf}

    {% include [terraform-definition](../../../_tutorials/terraform-definition.md) %}

    {% include [terraform-install](../../../_includes/terraform-install.md) %}

    Чтобы удалить метку функции:

    1. Откройте файл конфигурации {{ TF }} и в блоке `labels` удалите ненужную метку.

        Пример описания функции в конфигурации {{ TF }}:
      
        ```
        resource "yandex_function" "test-function" {
            name               = "test-function"
            description        = "Test function"
            user_hash          = "first-function"
            runtime            = "python37"
            entrypoint         = "main"
            memory             = "128"
            execution_timeout  = "10"
            service_account_id = "<идентификатор сервисного аккаунта>"
            tags               = ["my_tag"]
            labels = {
              <ключ1>:"<значение1>",
              <ключ2>:"<значение2>"
            }
            content {
                zip_filename = "<путь к ZIP-архиву>"
            }
        }
        ``` 

        Более подробную информацию о параметрах ресурса `yandex_function` см. в [документации провайдера]({{ tf-provider-resources-link }}/function).

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
      
    Проверить удаление меток функции можно с помощью команды [CLI](../../../cli/quickstart.md):

    ```
    yc serverless function get <имя_функции>|<идентификатор_функции>
    ```

- API {#api}

    Чтобы удалить метку функции, воспользуйтесь методом REST API [update](../../functions/api-ref/Function/update.md) для ресурса [Function](../../functions/api-ref/Function/index.md) или вызовом gRPC API [FunctionService/Update](../../functions/api-ref/grpc/function_service.md#Update).

{% endlist %}
