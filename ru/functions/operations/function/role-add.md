---
title: "Назначить роли на функцию"
description: "Следуя данной инструкции, вы сможете назначить роли на функцию."
---

# Назначить роли на функцию

{% list tabs group=instructions %}

- CLI {#cli}

    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

    Выполните команду, чтобы назначить [роль](../../security/index.md#roles-list) на функцию:

    * пользователю:
        ```
        yc serverless function add-access-binding \
          --id <идентификатор функции> \
          --user-account-id <идентификатор пользователя> \
          --role <роль>
        ```
        Результат:
        ```
        done (1s)
        ```
    * [сервисному аккаунту](../../../iam/concepts/users/service-accounts.md):  
        ```
        yc serverless function add-access-binding \
          --id <идентификатор функции> \
          --service-account-id <идентификатор сервисного аккаунта> \
          --role <роль>
        ```
        Результат:
        ```
        done (1s)
        ```
    * всем авторизованным пользователям (системная группа `allAuthenticatedUsers`):
        ```
        yc serverless function add-access-binding \
          --id <идентификатор функции> \
          --all-authenticated-users \
          --role <роль>
        ```
        Результат:
        ```
        done (1s)
        ```

- API {#api}

  Чтобы назначить роли на функцию, воспользуйтесь методом REST API [setAccessBindings](../../functions/api-ref/Function/setAccessBindings.md) для ресурса [Function](../../functions/api-ref/Function/index.md) или вызовом gRPC API [FunctionService/SetAccessBindings](../../functions/api-ref/grpc/function_service.md#SetAccessBindings).

{% endlist %}