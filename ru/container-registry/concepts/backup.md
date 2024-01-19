---
title: "Резервное копирование Docker-образов"
description: "Данные в {{ container-registry-name }} надежно хранятся и реплицируются в инфраструктуре {{ yandex-cloud }}, однако в любой момент вы можете сделать резервные копии Docker-образов. Чтобы выполнить резервное копирование Docker-образов, выполните инструкции описанные в данной статье."
---

# Резервное копирование Docker-образов в {{ container-registry-name }}

Данные в {{ container-registry-name }} надежно хранятся и реплицируются в инфраструктуре {{ yandex-cloud }}, однако в любой момент вы можете сделать резервные копии [Docker-образов](../concepts/docker-image.md).

Чтобы выполнить резервное копирование Docker-образов:

1. Посмотрите список имеющихся [реестров](registry.md):

   {% list tabs group=instructions %}

   - Консоль управления {#console}

     1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите посмотреть список реестров.
     1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_container-registry }}**.
     1. На вкладке **{{ ui-key.yacloud.cr.switch_registry }}** отображен список реестров.

   - CLI {#cli}

     {% include [cli-install](../../_includes/cli-install.md) %}
     
     {% include [default-catalogue](../../_includes/default-catalogue.md) %}

     ```bash
     yc container registry list
     ```

     Результат:

     ```bash
     +----------------------+-------------+----------------------+
     |          ID          |   NAME      |      FOLDER ID       |
     +----------------------+-------------+----------------------+
     | crp1abc2tl3d******** | my-registry | b1g23ga82bcv******** |
     +----------------------+-------------+----------------------+
     ```

   - API {#api}

     Воспользуйтесь методом [list](../api-ref/Registry/list.md) для ресурса [Registry](../api-ref/Registry/).

   {% endlist %}

1. Получите информацию об имеющихся Docker-образах по каждому реестру:

   {% list tabs group=instructions %}

   - Консоль управления {#console}

     1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором хотите посмотреть информацию о Docker-образах.
     1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_container-registry }}**.
     1. На вкладке **{{ ui-key.yacloud.cr.switch_registry }}** нажмите на имя нужного реестра.
     1. На странице **{{ ui-key.yacloud.cr.registry.overview.section_repositories }}** нажмите на имя нужного репозитория.
     1. На странице **{{ ui-key.yacloud.cr.registry.overview.section_images }}** отображены Docker-образы.

   - CLI {#cli}

     ```bash
     yc container image list --registry-name my-registry
     ```

     Где `registry-name`— имя реестра.

     Результат:

     ```bash
     +----------------------+---------------------+-----------------------------+-------+-----------------+
     |          ID          |       CREATED       |            NAME             | TAGS  | COMPRESSED SIZE |
     +----------------------+---------------------+-----------------------------+-------+-----------------+
     | crpabcpg1pde******** | 2020-10-26 15:45:39 | crp1abc2tl3d********/ubuntu | hello | 27.2 MB         |
     +----------------------+---------------------+-----------------------------+-------+-----------------+
     ```

   - API {#api}

     Воспользуйтесь методом [list](../api-ref/Image/list.md) для ресурса [Image](../api-ref/Image/).

   {% endlist %}

1. Скачайте необходимые Docker-образы из каждого реестра, используя полный адрес — `{{ registry }}/<имя>:<тег>`:

   {% list tabs group=instructions %}

   - CLI {#cli}

     ```bash
     docker pull {{ registry }}/crp1abc2tl3d********/ubuntu:hello
     ```

     Результат:

     ```bash
     hello: Pulling from crp1abc2tl3d********/ubuntu
     Digest: sha256:1d2b339619babc2d008eca2d5293e3c43ff84cbee597ff76dd3e7f7d********
     Status: Downloaded newer image for {{ registry }}/crp1abc2tl3d********/ubuntu:hello
     {{ registry }}/crp1abc2tl3d********/ubuntu:hello
     ```

   {% endlist %}

1. Проверьте, что Docker-образ действительно скачался:

   {% list tabs group=instructions %}

   - CLI {#cli}

     ```bash
     docker image list
     ```

     Результат:

     ```bash
     REPOSITORY                                   TAG     IMAGE ID      CREATED     SIZE
     {{ registry }} /crp1abc2tl3d********/ubuntu       hello   d12e********  3 days ago  72.9MB
     ubuntu                                       latest  d12e********  3 days ago  72.9MB
     ```

   {% endlist %}