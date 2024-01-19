### Подготовьте инфраструктуру {#deploy-infrastructure}

{% list tabs group=instructions %}

- Вручную {#manual}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Если у вас еще нет [сети](../../vpc/concepts/network.md#network), [создайте ее](../../vpc/operations/network-create.md).
  1. Если у вас еще нет [подсетей](../../vpc/concepts/network.md#subnet), [создайте их](../../vpc/operations/subnet-create.md) в [зонах доступности](../../overview/concepts/geo-scope.md), где будут созданы [кластер {{ managed-k8s-full-name }}](../../managed-kubernetes/concepts/index.md#kubernetes-cluster) и [группа узлов](../../managed-kubernetes/concepts/index.md#node-group).
  1. [Создайте сервисные аккаунты](../../iam/operations/sa/create.md):
     * Для ресурсов с [ролью](../../iam/concepts/access-control/roles.md) [{{ roles-editor }}](../../resource-manager/security/index.md#roles-list) на [каталог](../../resource-manager/concepts/resources-hierarchy.md#folder), в котором создается кластер {{ managed-k8s-name }}. От его имени будут создаваться ресурсы, необходимые кластеру {{ managed-k8s-name }}.
     * Для узлов с ролями [{{ roles-cr-puller }}](../../iam/concepts/access-control/roles.md#cr-images-puller) и [{{ roles-cr-pusher }}](../../iam/concepts/access-control/roles.md#cr-images-pusher) на каталог с [реестром](../../container-registry/concepts/registry.md) [Docker-образов](../../container-registry/concepts/docker-image.md). От его имени узлы {{ managed-k8s-name }} будут загружать в реестр собранные в {{ GL }} Docker-образы, а также скачивать их для запуска [подов](../../managed-kubernetes/concepts/index.md#pod).

     {% note tip %}

     Вы можете использовать один и тот же [сервисный аккаунт](../../iam/concepts/users/service-accounts.md) для обеих операций.

     {% endnote %}

  1. [Создайте кластер {{ managed-k8s-name }} ](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create.md#kubernetes-cluster-create) и [группу узлов](../../managed-kubernetes/operations/node-group/node-group-create.md). При создании кластера {{ managed-k8s-name }} укажите ранее созданные сервисные аккаунты для ресурсов и узлов.
  1. [Настройте группы безопасности](../../managed-kubernetes/operations/connect/security-groups.md) для работы кластера {{ managed-k8s-name }}.
  1. [Настройте группу безопасности по умолчанию](../../managed-gitlab/operations/connect.md) для работы [инстанса {{ mgl-name }}](../../managed-gitlab/concepts/index.md#instance).
  1. [Создайте реестр {{ container-registry-full-name }}](../../container-registry/operations/registry/registry-create.md).
  1. [Создайте авторизованный ключ](../../iam/operations/authorized-key/create.md) для сервисного аккаунта с ролью `{{ roles-cr-pusher }}` и сохраните в файл `key.json`:

     ```bash
     yc iam key create \
       --service-account-name <имя_сервисного_аккаунта> \
       --output key.json
     ```

     Ключ необходим для доступа к реестру из {{ GL }}.

- {{ TF }} {#tf}

  1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
  1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
  1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
  1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}

  1. Скачайте в ту же рабочую директорию файл конфигурации [k8s-and-registry-for-gitlab.tf](https://github.com/yandex-cloud/examples/blob/master/tutorials/terraform/managed-kubernetes/k8s-and-registry-for-gitlab.tf).

     В этом файле описаны:
     * [Сеть](../../vpc/concepts/network.md#network).
     * [Подсеть](../../vpc/concepts/network.md#subnet).
     * [Группа безопасности по умолчанию](../../vpc/concepts/security-groups.md) и правила, необходимые для работы [инстанса {{ mgl-name }}](../../managed-gitlab/concepts/index.md#instance).
     * [Группа безопасности](../../vpc/concepts/security-groups.md) и правила, необходимые для работы [кластера {{ managed-k8s-full-name }}](../../managed-kubernetes/concepts/index.md#kubernetes-cluster).
     * Кластер {{ managed-k8s-name }}.
     * [Сервисный аккаунт](../../iam/concepts/users/service-accounts.md), необходимый для работы кластера {{ managed-k8s-name }} и [группы узлов](../../managed-kubernetes/concepts/index.md#node-group).
     * [Реестр](../../container-registry/concepts/registry.md) [{{ container-registry-full-name }}](../../container-registry/).
     * [Авторизованный ключ](../../iam/concepts/authorization/key.md) для сервисного аккаунта. Ключ необходим для доступа к реестру из {{ GL }}.
     * Локальный файл `key.json` с данными авторизованного ключа.
  1. Укажите в файле `k8s-and-registry-for-gitlab.tf`:
     * [Идентификатор каталога](../../resource-manager/operations/folder/get-id.md).
     * [Версию {{ k8s }}](../../managed-kubernetes/concepts/release-channels-and-updates.md) для кластера {{ managed-k8s-name }} и групп узлов.
     * Имя сервисного аккаунта кластера {{ managed-k8s-name }}.
     * Имя реестра {{ container-registry-name }}.
  1. Проверьте корректность файлов конфигурации {{ TF }} с помощью команды:

     ```bash
     terraform validate
     ```

     Если в файлах конфигурации есть ошибки, {{ TF }} на них укажет.
  1. Создайте необходимую инфраструктуру:

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

     {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

{% note warning %}

Для реальных приложений доступ сервисных аккаунтов кластера {{ managed-k8s-name }} к загрузке Docker-образов в реестр должен быть ограничен по соображениям безопасности. В этом случае создайте отдельный сервисный аккаунт с ролью `{{ roles-cr-pusher }}` и указывайте его для развертывания приложений.

{% endnote %}