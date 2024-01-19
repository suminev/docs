---
title: "Инструкция по созданию подключения к {{ PG }} в {{ datalens-full-name }}"
description: "Из статьи вы узнаете, как подключиться к {{ PG }} в {{ datalens-full-name }}."
---

# Создание подключения к {{ PG }}


{% include [connection-note](../../../_includes/datalens/datalens-connection-note-ip.md) %}

Чтобы создать подключение к {{ PG }}:


{% include [datalens-workbooks-collections-note](../../../_includes/datalens/operations/datalens-workbooks-collections-note.md) %}



1. Перейдите на [страницу подключений]({{ link-datalens-main }}/connections).


1. Нажмите кнопку **Создать подключение**.
1. Выберите подключение **{{ PG }}**.


1. Выберите тип подключения:

   {% list tabs group=connection %}

   - Выбрать в каталоге {#catalog}

     {% include [datalens-db-select](../../../_includes/datalens/datalens-db-select.md) %}

     - **Кластер**. Укажите кластер из списка доступных кластеров {{ PG }}. В настройках кластера должна быть активирована опция **Доступ из {{ datalens-short-name }}**. Если у вас нет доступного кластера, нажмите кнопку **Создать новый**.

       {% include [datalens-cluster-list](../../../_includes/datalens/datalens-cluster-list.md) %}

     * **Имя хоста**. Выберите имя хоста из списка доступных в кластере {{ PG }}. Вы можете выбрать несколько хостов. Если к первому хосту подключиться не получится, {{ datalens-short-name }} выберет следующий из списка.
     * **Порт**. Укажите порт подключения к {{ PG }}. В {{ yandex-cloud }} по умолчанию используется порт 6432.
     * **Путь к базе данных**. Укажите имя подключаемой базы данных.
     * **Имя пользователя**. Укажите имя пользователя для подключения к {{ PG }}.
     * **Пароль**. Укажите пароль для пользователя.
     * **Время жизни кеша в секундах**. Укажите время жизни кеша или оставьте значение по умолчанию. Рекомендованное значение — 300 секунд (5 минут).
     * **Уровень доступа SQL запросов**. Позволяет использовать произвольный SQL-запрос для [формирования датасета](../../concepts/dataset/settings.md#sql-request-in-datatset).

   - Указать вручную {#manual}

     {% include [datalens-db-specify-mannualy](../../../_includes/datalens/datalens-db-specify-mannualy.md) %}

     {% include [datalens-db-connection-parameters-postgresql](../../../_includes/datalens/datalens-db-connection-parameters-postgresql.md) %}

   {% endlist %}


1. Нажмите кнопку **Создать подключение**.
1. Укажите название подключения и нажмите кнопку **Создать**.

{% include [datalens-check-host](../../../_includes/datalens/operations/datalens-check-host.md) %}



## Дополнительные настройки {#postgresql-additional-settings}

Вы можете указать дополнительные параметры подключения в разделе **Продвинутые настройки подключения**:

* **Указание collate в запросе** — если в запросах к БД требуется явно задать правило сортировки, выберите один из режимов:

  * **Авто** — применяется настройка по умолчанию, решение о включении локали `en_US` остается на усмотрение {{ datalens-short-name }}.
  * **Вкл** — применяется настройка {{ datalens-short-name }}, для отдельных выражений в запросе указывается локаль `en_US`. Это приводит к использованию соответствующей логики сортировки сервером, вне зависимости от настроек сервера и конкретных таблиц. Используйте настройку {{ datalens-short-name }}, если локаль в БД несовместима с работой {{ datalens-short-name }}. Подробнее о локали и `LC_COLLATE` смотрите в разделе [{#T}](../../../managed-postgresql/operations/cluster-create.md#create-cluster).
  * **Выкл** — применяется настройка по умолчанию, {{ datalens-short-name }} использует только имеющиеся параметры локали БД.

* **TLS** — опция определяет необходимость использования протокола TLS. Когда опция включена, параметр `sslmode` принимает значение `required`, когда выключена — `prefer`.
* **CA Certificate** — чтобы загрузить сертификат, нажмите кнопку **Прикрепить файл** и укажите файл сертификата. Когда сертификат загружен, поле отображает название файла.
* {% include [datalens-db-connection-export-settings-item](../../../_includes/datalens/operations/datalens-db-connection-export-settings-item.md) %}
