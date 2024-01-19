---
title: "Удаление кластера {{ AF }}"
description: "Вы можете удалить кластер {{ AF }}, если он вам больше не нужен."
keywords:
  - удаление кластера {{ AF }}
  - кластер {{ AF }}
  - {{ AF }}
  - Airflow
---

# Удаление кластера {{ AF }}

Вы можете удалить кластер {{ AF }}, если он вам больше не нужен.

## Удалить кластер {#delete}

Перед удалением кластера отключите защиту от удаления для кластера, если она включена.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, из которого нужно удалить кластер.
  1. Выберите сервис **{{ maf-name }}**.
  1. Нажмите значок ![image](../../_assets/console-icons/ellipsis.svg) для нужного кластера и выберите пункт **{{ ui-key.yacloud.mdb.cluster.overview.button_action-delete }}**.
  1. В открывшемся окне подтвердите удаление и нажмите кнопку **{{ ui-key.yacloud.mdb.dialogs.popup-confirm-delete-cluster_button }}**.

{% endlist %}
