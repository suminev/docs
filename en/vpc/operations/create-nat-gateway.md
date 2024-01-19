---
title: "How to create and set up a NAT gateway in {{ vpc-full-name }}"
description: "Follow this guide to create and set up a NAT gateway."
---

# Setting up a NAT gateway

To create and set up a NAT gateway:

{% list tabs %}

- Management console

   1. In the [management console]({{ link-console-main }}), go to the folder where you need to create a gateway.
   1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.vpc.switch_gateways }}**.
   1. Click **{{ ui-key.yacloud.common.create }}**.
   1. Enter a name for the gateway. The naming requirements are as follows:

      {% include [name-format](../../_includes/name-format.md) %}

   1. (Optional) Add a gateway description.
   1. The default gateway type is `{{ ui-key.yacloud.vpc.gateways.value_gateway-type-egress-nat }}`.
   1. Click **{{ ui-key.yacloud.common.save }}**.
   1. In the left-hand panel, select **{{ ui-key.yacloud.vpc.network.switch_route-table }}**.
   1. Click **{{ ui-key.yacloud.common.create }}** to add a new table, or select an existing one.
   1. Click **{{ ui-key.yacloud.vpc.route-table-form.label_add-static-route }}**.
   1. In the window that opens, select `{{ ui-key.yacloud.vpc.add-static-route.value_gateway }}` in the **{{ ui-key.yacloud.vpc.add-static-route.field_next-hop-address }}** field.
   1. In the **{{ ui-key.yacloud.vpc.add-static-route.value_gateway }}** field, select the NAT gateway you created. The destination prefix will be propagated automatically.
   1. Click **{{ ui-key.yacloud.vpc.add-static-route.button_add }}**.
   1. Click **{{ ui-key.yacloud.vpc.route-table.edit.button_edit }}**.

   Next, link the route table to a subnet to route traffic from it via the NAT gateway:

   1. In the left-hand panel, select ![image](../../_assets/vpc/subnets.svg) **{{ ui-key.yacloud.vpc.switch_networks }}**.
   1. In the required subnet row, click ![image](../../_assets/options.svg).
   1. In the menu that opens, select **{{ ui-key.yacloud.vpc.subnetworks.button_action-add-route-table }}**.
   1. In the window that opens, select the created table from the list.
   1. Click **{{ ui-key.yacloud.vpc.subnet.add-route-table.button_add }}**.

- CLI

   {% include [include](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   1. View a description of the CLI create gateway command:

      ```bash
      yc vpc gateway create --help
      ```

   1. Create a gateway in the default folder:

      ```bash
      yc vpc gateway create \
         --name test-gateway
      ```
   1. Get the gateway ID:

      ```bash
      yc vpc gateway list
      ```

      Command result:

      ```text
      +----------------------+--------------+-------------+
      |          ID          |     NAME     | DESCRIPTION |
      +----------------------+--------------+-------------+
      | enpkq1v2e7p0******** | test-gateway |             |
      +----------------------+--------------+-------------+
      ```

   1. Create a route table with the gateway as the next hop and the `0.0.0.0/0` destination prefix:

      ```bash
      yc vpc route-table create \
        --name=test-route-table \
        --network-name=<network_name> \
        --route destination=0.0.0.0/0,`
                gateway-id=enpkq1v2e7p0********
      ```

      Where `--network-name` is the name of the network in which you are creating the table.

   1. Link the table to the subnet:

      ```bash
      yc vpc subnet update <subnet_name> \
        --route-table-name=test-route-table
      ```

- {{ TF }}

   {% include [terraform-definition](../../_tutorials/terraform-definition.md) %}

   {% include [terraform-install](../../_includes/terraform-install.md) %}

   To create a NAT gateway, specify it as the next hop in the [route](../concepts/static-routes.md) table, and link the table to the subnet, use the following configuration:

   ```hcl
   data "yandex_vpc_network" "net" {
     folder_id = "<folder_ID>"
     name      = "<network_name>"
   }

   resource "yandex_vpc_subnet" "subnet" {
     folder_id      = "<folder_ID>"
     name           = "<subnet_name>"
     v4_cidr_blocks = "10.20.30.0/24"
     zone           = {{ region-id }}-a
     network_id     = data.yandex_vpc_network.net.id
     route_table_id = yandex_vpc_route_table.rt.id
   }

   resource "yandex_vpc_gateway" "nat_gateway" {
     name = "test-gateway"
     shared_egress_gateway {}
   }

   resource "yandex_vpc_route_table" "rt" {
     name       = test-route-table
     network_id = "<network_ID>"

     static_route {
       destination_prefix = "0.0.0.0/0"
       gateway_id         = yandex_vpc_gateway.nat_gateway.id
     }
   }
   ```

   Where `folder_id` is the ID of the folder hosting the subnet.

- API

   1. Create a NAT gateway. Use the [create](../api-ref/Gateway/create.md) REST API method for the [Gateway](../api-ref/Gateway/index.md) resource or the [GatewayService/Create](../api-ref/grpc/gateway_service.md#Create) gRPC API call, and provide the following in the request:

      * ID of the folder where the gateway will be placed, in the `folderId` parameter.
      * Gateway name, in the `name` parameter. The name format is as follows:

         {% include [name-format](../../_includes/name-format.md) %}

   1. Link the NAT gateway to the new routing table. For this, use the [create](../api-ref/RouteTable/create.md) REST API method for the [RouteTable](../api-ref/RouteTable/index.md) resource or the [RouteTableService/Create](../api-ref/grpc/route_table_service.md#Create) gRPC API call, and provide the following in the request:

      * ID of the folder where the route table will be placed, in the `folderId` parameter.
      * Route table name, in the `name` parameter. The name format is as follows:

         {% include [name-format](../../_includes/name-format.md) %}
      * ID of the network where the route table will be placed, in the `networkId` parameter.
      * `0.0.0.0/0` as the destination subnet prefix, in the `staticRoutes[].destinationPrefix` parameter.
      * NAT gateway name, in the `staticRoutes[].gatewayId` parameter.

         {% include [get-nat-gateway](../../_includes/vpc/get-nat-gateway.md) %}

   1. Link the route table to the subnet. For this, use the [update](../api-ref/Subnet/update.md) REST API method for the [Subnet](../api-ref/Subnet/index.md) resource or the [SubnetService/Update](../api-ref/grpc/subnet_service.md#Update) gRPC API call, and provide the following in the request:

      * Subnet ID, in the `subnetId` parameter.

         {% include [get-subnet-id](../../_includes/vpc/get-subnet-id.md) %}

         {% include [get-catalog-id](../../_includes/get-catalog-id.md) %}

      * Route table ID, in the `routeTableId` parameter.
      * Name of the `routeTableId` parameter, in the `updateMask` parameter.

      {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}

{% endlist %}
