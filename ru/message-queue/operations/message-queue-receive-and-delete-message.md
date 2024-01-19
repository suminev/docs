---
title: "Как принимать и удалять сообщения в {{ message-queue-full-name }}"
description: "Следуя данной инструкции, вы сможете принимать и удалять сообщения." 
---

# Прием и удаление сообщений

Отправленные в очередь {{ message-queue-name }} сообщения получают потребители — компоненты распределенных приложений. После приема и обработки сообщений их следует удалять из очереди.

{% list tabs group=instructions %}

- AWS CLI {#cli}
  
  1. [Установите и настройте](configuring-aws-cli.md) AWS CLI.
  1. [Создайте](message-queue-new-queue.md) очередь сообщений и [отправьте](message-queue-send-message.md) в нее тестовое сообщение.
  1. Примите отправленное ранее сообщение:

      ```bash
      aws sqs receive-message \
        --queue-url <URL_очереди> \
        --endpoint <эндпоинт>
      ```
      
      Где:

      * `--queue-url` — URL очереди, из которой нужно получить сообщение.
      * `--endpoint` — эндпоинт в значении `https://message-queue.{{ api-host }}/`.

      Результат:

      ```json
      {
          "Messages": [
              {
                  "MessageId": "948de7-9ec8d787-c*******-*",
                  "ReceiptHandle": "EAEggbj********",
                  "MD5OfBody": "ed076287532e86365e841e92********",
                  "Body": "Hello World",
                  "Attributes": {
                      "ApproximateFirstReceiveTimestamp": "15459********",
                      "ApproximateReceiveCount": "1",
                      "SentTimestamp": "15459********",
                      "SenderId": "abcdefkbh72is78********"
                  }
              }
          ]
      }
      ```

      Сохраните значение параметра `ReceiptHandle`, оно понадобится позднее.

      {% note info %}

      Если в очереди нет сообщений, команда `aws sqs receive-message` завершится без текстового вывода в терминал.

      {% endnote %}

  1. Для удаления сообщения из очереди используйте [идентификатор получения](../concepts/message.md) — значение параметра `ReceiptHandle`, полученного вместе с сообщением. 

     Удалите полученное сообщение из очереди:

      ```bash
      aws sqs delete-message \
        --queue-url <URL_очереди> \
        --endpoint <эндпоинт> \
        --receipt-handle <идентификатор_получения>
      ```
    
      Где:

      * `--queue-url` — URL очереди, из которой нужно удалить сообщение.
      * `--endpoint` — эндпоинт в значении `https://message-queue.{{ api-host }}/`.
      * `--receipt-handle` — сохраненный ранее идентификатор получения сообщения `ReceiptHandle`.

{% endlist %}

Успешное выполнение команды удаления не сопровождается текстовым выводом в терминал. Чтобы убедиться в удалении сообщения:

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором создана очередь сообщений.
  1. В списке сервисов выберите **{{ message-queue-name }}**.
  1. Текущее количество сообщений в очереди указано в поле **Сообщений в очереди**.

{% endlist %}