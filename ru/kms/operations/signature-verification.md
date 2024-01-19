---
title: "Как создать электронную подпись"
description: "Следуя данной инструкции, вы сможете создать электронную подпись."
---

# Электронная подпись и проверка подписи по хэшу данных

{% note info %}

{% include [preview-pp.md](../../_includes/preview-pp.md) %}

{% endnote %}

Сервис {{ kms-name }} позволяет создать [электронную подпись](../concepts/digital-signature.md), которую можно использовать для подтверждения подлинности и проверки целостности данных, а также для защиты подписанных данных от изменений.

## Перед началом работы {#before-you-begin}

Проверка действительности электронной подписи в этой инструкции выполняется с помощью утилиты [OpenSSL](https://www.openssl.org/). Если эта утилита у вас не установлена, установите ее:

{% include [install-openssl](../../_includes/kms/install-openssl.md) %}

## Создайте электронную подпись {#create-signature}

В зависимости от размера подписываемого сообщения или файла сервис {{ kms-short-name }} позволяет создавать подпись сообщения [на приватном ключе](#message-signing) или подпись [по хэшу данных](#hash-signing).

### Подпись сообщения на приватном ключе {#message-signing}

{% note info %}

Подпись на приватном ключе применяется для сообщений размером до 32 КБ.

{% endnote %}

1. Если у вас еще нет ключевой пары электронной подписи, [создайте](./asymmetric-signature-key.md#create) ее.

1. Получите открытый ключ подписи и сохраните его:

    {% list tabs group=instructions %}

    - Консоль управления {#console}
    
      1. В [консоли управления]({{ link-console-main }}) выберите [каталог](../../resource-manager/concepts/resources-hierarchy.md#folder), в котором находится нужная ключевая пара электронной подписи.
      1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_kms }}**.
      1. На панели слева выберите ![image](../../_assets/kms/asymmetric-key.svg) **{{ ui-key.yacloud.kms.switch_asymmetric-keys }}**.
      1. Перейдите на вкладку **{{ ui-key.yacloud.kms.asymmetric-key.form.label_signature }}**.
      1. В строке с нужной ключевой парой нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) и выберите **{{ ui-key.yacloud.kms.asymmetric-keys.action_public-key }}**.
      1. В открывшемся окне нажмите кнопку **{{ ui-key.yacloud.kms.asymmetric-keys.button_download }}**, чтобы скачать публичный ключ электронной подписи.

    - CLI {#cli}

      {% include [cli-install](../../_includes/cli-install.md) %}

      {% include [default-catalogue](../../_includes/default-catalogue.md) %}
  
      1. Посмотрите описание команды CLI для получения открытого ключа подписи:

          ```bash
          yc kms asymmetric-signature-crypto get-public-key --help
          ```

      1. [Получите](../../resource-manager/operations/folder/get-id.md) идентификатор каталога, в котором сохранена ключевая пара электронной подписи.

      1. {% include [get-signature-key](../../_includes/kms/get-signature-key.md) %}
      
      1. {% include [cli-get-public-key](../../_includes/kms/cli-get-public-key.md) %}

    {% endlist %}

1. Создайте файл с сообщением в кодировке `base64`:

    1. Создайте текстовый файл, например, `message.txt`:

        ```bash
        cat > message.txt
        My sample message.
        It will be used to verify ECDSA signature.
        ```

        Размер сообщения не должен превышать 32 КБ.  

    1. Переведите сообщение в кодировку `base64`, указав путь к создаваемому файлу сообщения в `base64`:

        ```bash
        base64 message.txt > <файл_сообщения_в_base64>
        ```

1. Создайте подпись сообщения:

    {% list tabs group=instructions %}

    - CLI {#cli}

      1. Посмотрите описание команды CLI для получения электронной подписи:

          ```bash
          yc kms asymmetric-signature-crypto sign --help
          ```

      1. Получите электронную подпись сообщения:
      
          ```bash
          yc kms asymmetric-signature-crypto sign \
            --id <идентификатор_ключевой_пары> \
            --signature-output-file <путь_к_файлу_подписи> \
            --message-file <путь_к_файлу_с_сообщением> \
            --inform base64 \
            --outform base64
          ```
          
          Где:
          * `--id` — идентификатор ключевой пары электронной подписи.
          * `--signature-output-file` — путь к файлу, в который будет сохранена электронная подпись.
          * `--message-file` — путь к созданному ранее файлу с сообщением в кодировке `base64`.
          * `--inform` — формат файла с сообщением. Возможные значения: `raw` (по умолчанию), `base64` и `hex`.
          * `--outform` — формат файла подписи. Возможные значения: `raw` (по умолчанию), `base64` и `hex`.

          Результат:

          ```bash
          key_id: abjcg4mhmdfe********
          signature: MAa7C...imw==
          ```

      1. Переведите полученную электронную подпись в формат [DER](https://ru.wikipedia.org/wiki/X.690#Distinguished_Encoding_Rules), который необходим для утилиты `OpenSSL`:

          ```bash
          echo -n "$(< <путь_к_файлу_подписи>)" | base64 -d > <файл_подписи>
          ```
          
          Где:
          * `<путь_к_файлу_подписи>` — путь к полученному на предыдущем шаге файлу подписи.
          * `<файл_подписи>` — путь к создаваемому файлу подписи в формате `DER`. 

      Полученный файл подписи в формате `DER` можно использовать при [проверке](#verify-digital-signature) подписи утилитой `OpenSSL`.

    {% endlist %}

## Подпись файла по хэшу данных {#hash-signing}

{% note info %}

Подпись по хэшу применяется для сообщений или файлов размером более 32 КБ.

{% endnote %}

1. Если у вас еще нет ключевой пары электронной подписи, [создайте](./asymmetric-signature-key.md#create) ее.

1. Получите открытый ключ электронной подписи и сохраните его:

    {% list tabs group=instructions %}

    - Консоль управления {#console}
    
      1. В [консоли управления]({{ link-console-main }}) выберите [каталог](../../resource-manager/concepts/resources-hierarchy.md#folder), в котором находится нужная ключевая пара электронной подписи.
      1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_kms }}**.
      1. На панели слева выберите ![image](../../_assets/kms/asymmetric-key.svg) **{{ ui-key.yacloud.kms.switch_asymmetric-keys }}**.
      1. Перейдите на вкладку **{{ ui-key.yacloud.kms.asymmetric-key.form.label_signature }}**.
      1. В строке с нужной ключевой парой нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) и выберите **{{ ui-key.yacloud.kms.asymmetric-keys.action_public-key }}**.
      1. В открывшемся окне нажмите кнопку **{{ ui-key.yacloud.kms.asymmetric-keys.button_download }}**, чтобы скачать публичный ключ подписи.

    - CLI {#cli}

      {% include [cli-install](../../_includes/cli-install.md) %}

      {% include [default-catalogue](../../_includes/default-catalogue.md) %}
  
      1. Посмотрите описание команды CLI для получения открытого ключа подписи:

          ```bash
          yc kms asymmetric-signature-crypto get-public-key --help
          ```

      1. [Получите](../../resource-manager/operations/folder/get-id.md) идентификатор каталога, в котором сохранена ключевая пара электронной подписи.

      1. {% include [get-signature-key](../../_includes/kms/get-signature-key.md) %}
      
      1. {% include [cli-get-public-key](../../_includes/kms/cli-get-public-key.md) %}

    {% endlist %}

1. Получите хэш файла:

    {% list tabs group=programming_language %}

    - Bash {#bash}
    
      Выполните команду:
      
      ```bash
      echo -n \
        $(<алгоритм_хэширования> <путь_к_исходному_файлу> | cut -d " " -f 1) > \
        <путь_к_файлу_с_хэшем>
      ```
      
      Где:
      * `<алгоритм_хэширования>` — алгоритм хэширования, использованный при создании ключевой пары электронной подписи. Алгоритм хэширования указан выше в поле `SIGNATURE ALGORITHM` в результатах получения списка ключевых пар. Возможные значения:
          * `sha256sum` — для алгоритмов SHA-256;
          * `sha384sum` — для алгоритмов SHA-384;
          * `sha512sum` — для алгоритмов SHA-512.
      * `<путь_к_исходному_файлу>` — путь к файлу, хэш которого нужно получить.
      * `<путь_к_файлу_с_хэшем>` — путь к файлу, в который будет сохранен хэш.

    - PowerShell {#powershell}
    
      Выполните команду:
      
      ```powershell
      (Get-FileHash -Path <путь_к_исходному_файлу> -Algorithm <алгоритм_хэширования>).Hash.ToLower() | `
        Out-File -FilePath <путь_к_файлу_с_хэшем> `
        -encoding ASCII `
        -NoNewline
      ```
      
      Где:
      * `<алгоритм_хэширования>` — алгоритм хэширования, использованный при создании ключевой пары подписи. Алгоритм хэширования указан выше в поле `SIGNATURE ALGORITHM` в результатах получения списка ключевых пар. Возможные значения:
          * `SHA256` — для алгоритмов SHA-256;
          * `SHA384` — для алгоритмов SHA-384;
          * `SHA512` — для алгоритмов SHA-512.
      * `<путь_к_исходному_файлу>` — путь к файлу, хэш которого нужно получить.
      * `<путь_к_файлу_с_хэшем>` — путь к файлу, в который будет сохранен хэш.
    
    В результате будет создан текстовый файл, содержащий хэш исходного файла.
    
    {% endlist %}

1. Создайте подпись файла по хэшу:

    {% list tabs group=instructions %}

    - CLI {#cli}

      1. Посмотрите описание команды CLI для получения электронной подписи по хэшу:

          ```bash
          yc kms asymmetric-signature-crypto sign-hash --help
          ```

      1. [Получите](../../resource-manager/operations/folder/get-id.md) идентификатор каталога, в котором сохранена ключевая пара электронной подписи.

      1. {% include [get-signature-key](../../_includes/kms/get-signature-key.md) %}

      1. Получите электронную подпись по хэшу:
      
          ```bash
          yc kms asymmetric-signature-crypto sign-hash \
            --id <идентификатор_ключевой_пары> \
            --signature-output-file <путь_к_файлу_подписи> \
            --message-hash-file <путь_к_файлу_с_хэшем> \
            --inform hex
          ```
          
          Где:
          * `--id` — идентификатор ключевой пары электронной подписи.
          * `--signature-output-file` — путь к файлу, в который будет сохранена электронная подпись.
          * `--message-hash-file` — путь к созданному ранее файлу с хэшем.
          * `--inform` — формат файла с хэшем. Для универсальности в примере приведено значение `hex` — такой формат работает на всех платформах. Возможные значения: `raw` (по умолчанию), `base64` и `hex`. 

          Результат:

          ```bash
          signature: W7V8A...22g==
          ```

    {% endlist %}

## Проверьте электронную подпись {#verify-digital-signature}

### Подпись на эллиптических кривых (ECDSA) {#ecdsa-verification}

{% include [signature-verification-ecdsa](../../_includes/kms/signature-verification-ecdsa.md) %}

### Подпись RSA {#rca-verification}

{% include [signature-verification-rsa](../../_includes/kms/signature-verification-rsa.md) %}
