# Как сменить внутренний IP-адрес


## Описание сценария {#case-description}

Необходимо сменить внутренний IP-адрес виртуальной машины.

## Решение {#case-resolution}

Изменить внутренний адрес виртуальной машины можно с помощью CLI, Terraform и API. 

{% list tabs %}

- CLI

    - [Установка и настройка CLI](../../../cli/quickstart);
    - [Команда](../../../cli/cli-ref/managed-services/compute/instance/update-network-interface) `yc compute instance update-network-interface`.
    Пример команды:
    ```
    yc compute instance update-network-interface <ID ВМ> --network-interface-index=0 --ipv4-address=<новый IP из диапазона подсети>
    ```
    Флаг `--network-interface-index` отвечает за номер обновляемого сетевого интерфейса, нумерация начинается с нуля.

- Terraform

   - [Параметр `ip_address`, блок `network_interface`, ресурс `yandex_compute_instance`](https://registry.tfpla.net/providers/yandex-cloud/yandex/latest/docs/resources/compute_instance#ip_address)


- API

    - [REST](../../../compute/api-ref/Instance/updateNetworkInterface);
    - [gRPC](../../../compute/api-ref/grpc/instance_service#UpdateNetworkInterface).

    Выполните команду ```yc compute instance update-network-interface <идентификатор_вм> --network-interface-index=0 --ipv4-address=<новый_IP_адрес>```.

{% endlist %}

{% note info %}

Сменить адрес можно только на остановленной машине.

{% endnote %}