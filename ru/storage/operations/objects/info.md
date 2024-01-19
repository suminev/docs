---
title: "Инструкция по получению информации об объекте {{ objstorage-full-name }}"
description: "Из этой статьи вы узнаете, как получить информацию об объекте {{ objstorage-full-name }}."
---

# Получение информации об объекте

{% list tabs group=instructions %}

- Консоль управления {#console}
  
  1. В [консоли управления]({{ link-console-main }}) выберите каталог.
  1. Выберите сервис **{{ objstorage-name }}**.
  1. Нажмите на имя необходимого бакета.
  1. Нажмите на имя объекта.

- API {#api}

  Чтобы получить информацию об объекте, воспользуйтесь методом S3 API [getObjectMeta](../../s3/api-ref/object/getobjectmeta.md).

{% endlist %}
