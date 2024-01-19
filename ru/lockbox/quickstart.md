---
title: "Начало работы с {{ lockbox-full-name }}"
description: "В этой инструкции вы научитесь создавать секреты и получать их содержимое с помощью сервиса хранения секретов {{ lockbox-full-name }}. Секрет состоит из набора версий, в которых хранятся ваши данные. Версия содержит наборы ключей и значений. Ключ — это несекретное название для значения, по которому вы будете его идентифицировать. Значение — это секретные данные."
---

# Начало работы с {{ lockbox-name }}

Создайте ваш первый секрет и получите его содержимое.

Секрет состоит из набора версий, в которых хранятся ваши данные. Версия содержит наборы ключей и значений: 
* Ключ — это несекретное название для значения, по которому вы будете его идентифицировать.
* Значение — это секретные данные.

Версия не изменяется. Для любого изменения количества пар ключей-значений или их содержимого необходимо создать новую версию.

## Создать секрет {#secret}

{% include [secret-create](../_includes/lockbox/secret-create.md) %}

## Получите содержимое секрета {#get}

### Роли, необходимые для получения секрета {#roles}

{% include [lockbox-and-kms-roles](../_includes/lockbox/lockbox-and-kms-roles.md) %}

### Получите содержимое секрета {#payload}

Для получения содержимого секрета обратитесь к нему с помощью [API](../glossary/rest-api.md). При запросе без указания версии, будет возвращаться содержимое текущей (последней) версии.

Вы можете использовать эту логику в скриптах, сервисах и приложениях, где необходимо использовать содержимое секрета.

{% list tabs group=instructions %}

- CLI {#cli}
  
  {% include [cli-install](../_includes/cli-install.md) %}
  
  {% include [default-catalogue](../_includes/default-catalogue.md) %}

  1. Посмотрите описание команды CLI для получения содержимого секрета:

      ```bash
      yc lockbox payload get --help
      ```
    
  1. {% include [get-secret-id](../_includes/lockbox/get-secret-id.md) %}
  
  1. Выполните команду:
      
      ```bash
      yc lockbox payload get \
        --id <идентификатор_секрета> \
        --key <ключ_секрета> \
        --version-id <идентификатор_версии_секрета>
      ```

      Где:

      * `--id` — идентификатор секрета. Обязательный параметр.
      * `--key` – ключ содержимого секрета для получения единственного значения. Необязательный параметр.
      * `--version-id` – версия секрета. Необязательный параметр. По умолчанию используется текущая версия секрета.
      
      Пример команды для получения содержимого секрета:

      ```bash
      yc lockbox payload get \
        --id e6qetpqfe8vv******** \
        --version-id e6qqr7k79ecm********
      ```

      В этом примере возвращается содержимое секрета версии `e6qqr7k79ecm********`.
      
      Результат:

      ```bash
      version_id: e6qqr7k79ecm********
      entries:
        - key: first_key
          text_value: value_1
        - key: second_key
          text_value: value_2
      ```

- API {#api}

    Чтобы получить содержимое секрета:
    1. [Получите IAM-токен](../iam/operations/iam-token/create.md), необходимый для аутентификации и сохраните его в переменную:
    
        ```
        export IAM_TOKEN=$(yc iam create-token)
        ```
        
        Также вы можете получить IAM-токен сервисного аккаунта изнутри [ВМ](../glossary/vm.md), к которой он привязан. Для этого выполните запрос в [сервис метаданных](../compute/operations/vm-info/get-info.md#request-examples). Пример с использованием утилиты [jq](https://stedolan.github.io/jq/):
        
        ```                
        export IAM_TOKEN=$(curl -H Metadata-Flavor:Google http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token | jq -r .access_token)
        ```
       
    1. Выполните запрос: 
    
        ```
        curl -X GET -H "Authorization: Bearer ${IAM_TOKEN}" \
        https://{{ api-host-lockbox-payload }}/lockbox/v1/secrets/<идентификатор_секрета>/payload
        ```
{% endlist %}
