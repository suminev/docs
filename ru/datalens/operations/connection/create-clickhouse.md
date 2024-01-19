---
title: "Как создать подключение к {{ CH }}"
description: "Следуя данной инструкции, вы сможете создать подключение к {{ CH }}."
---

# Создание подключения к {{ CH }}

{% note info %}

Все запросы к данным выполняются с включенным флагом [join_use_nulls]({{ ch.docs }}/operations/settings/settings/#join_use_nulls). Ознакомьтесь с разделом [{#T}](#ch-connection-specify), если вы используете представления (VIEW) или подзапросы с секцией JOIN в {{ datalens-short-name }}.

{% endnote %}


{% include [connection-note](../../../_includes/datalens/datalens-connection-note-ip.md) %}

Чтобы создать подключение к {{ CH }}:


{% include [datalens-workbooks-collections-note](../../../_includes/datalens/operations/datalens-workbooks-collections-note.md) %}



1. Перейдите на [страницу подключений]({{ link-datalens-main }}/connections).


1. Нажмите кнопку **Создать подключение**.
1. Выберите подключение **{{ CH }}**.


1. Выберите тип подключения:

   {% list tabs group=connection %}

   - Выбрать в каталоге {#catalog}

     {% include [datalens-db-select](../../../_includes/datalens/datalens-db-select.md) %}

     * **Кластер**. Укажите кластер из списка доступных кластеров {{ CH }}. В настройках кластера должна быть активирована опция **Доступ из {{ datalens-short-name }}**. Если у вас нет доступного кластера, нажмите кнопку **Создать новый**.

       {% include [datalens-cluster-list](../../../_includes/datalens/datalens-cluster-list.md) %}

     * **Имя хоста**. Выберите имя хоста из списка доступных в кластере {{ CH }}. Вы можете выбрать несколько хостов. Если к первому хосту подключиться не получится, {{ datalens-short-name }} выберет следующий из списка.
     * **Порт HTTP-интерфейса**. Укажите порт подключения к {{ CH }}. Порт по умолчанию — 8443.
     * **Имя пользователя**. Укажите имя пользователя для подключения к {{ CH }}.

       {% include [datalens-db-note](../../../_includes/datalens/datalens-db-note.md) %}

     * **Пароль**. Укажите пароль для пользователя.
     * **Время жизни кеша в секундах**. Укажите время жизни кеша или оставьте значение по умолчанию. Рекомендованное значение — 300 секунд (5 минут).
     * **Уровень доступа SQL запросов**. Позволяет использовать произвольный SQL-запрос для [формирования датасета](../../concepts/dataset/settings.md#sql-request-in-datatset).

   - Указать вручную {#manual}

     {% include [datalens-db-specify-mannualy](../../../_includes/datalens/datalens-db-specify-mannualy.md) %}

     {% include [datalens-db-connection-parameters](../../../_includes/datalens/datalens-db-connection-parameters.md) %}

   {% endlist %}


1. Нажмите кнопку **Создать подключение**.
1. Укажите название подключения и нажмите кнопку **Создать**.

{% include [datalens-check-host](../../../_includes/datalens/operations/datalens-check-host.md) %}



## Дополнительные настройки {#clickhouse-additional-settings}

Вы можете указать дополнительные параметры подключения в разделе **Продвинутые настройки подключения**:

* **TLS** — когда опция включена, при взаимодействии с БД используется протокол `HTTPS`, когда выключена — `HTTP`.

* **CA Certificate** — чтобы загрузить сертификат, нажмите кнопку **Прикрепить файл** и укажите файл сертификата. Когда сертификат загружен, поле отображает название файла.

* {% include [datalens-db-connection-export-settings-item](../../../_includes/datalens/operations/datalens-db-connection-export-settings-item.md) %}

## Особенности работы с подключением к {{ CH }} {#ch-connection-specify}

Вы можете создавать датасеты поверх представлений (`VIEW`) в {{ CH }}, содержащих секцию `JOIN`. Для этого представление должно быть создано с включенной опцией `join_use_nulls`. Рекомендуется выставлять настройку `join_use_nulls = 1` в секции `SETTINGS`:

```sql
CREATE VIEW ... (
    ...
) AS
    SELECT
        ...
    FROM
        ...
    SETTINGS join_use_nulls = 1
```

Также следует включать эту опцию для подзапросов raw-sql, которые используются как источник данных в датасете.

Чтобы избежать ошибок при работе с представлениями в {{ datalens-short-name }}, содержащими секцию JOIN, создайте заново все представления с настройкой `join_use_nulls = 1`. Пустые ячейки при этом заполнятся значениями `NULL`, а тип соответствующих полей преобразуется в [Nullable]({{ ch.docs }}/sql-reference/data-types/nullable/#data_type-nullable).

{% include [clickhouse-disclaimer](../../../_includes/clickhouse-disclaimer.md) %}
