---
title: "How to get information about a sink in {{ cloud-logging-full-name }}"
description: "Follow this guide to get information about a sink."
---

# Getting information about a sink

{% list tabs %}

- Management console

   1. In the [management console]({{ link-console-main }}), select the folder where your log sink is located.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_logging }}**.
   1. In the left-hand panel, select ![image](../../_assets/console-icons/folder-arrow-down.svg) **{{ ui-key.yacloud.logging.label_sinks }}**.
   1. Select a sink.
   1. The **{{ ui-key.yacloud.common.overview }}** page will show the sink details.

- CLI

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To find out the ID or name of a log sink, [get](list-sink.md) a list of sinks in the folder.

   To get information about a sink, run this command:

   ```bash
   yc logging sink get <sink_name>
   ```

   Result:

   ```bash
   id: e23e73lehbfv********
   folder_id: b1gmit33ngp3********
   cloud_id: b1gia87mbaom********
   created_at: "2023-11-07T09:22:28.946Z"
   name: test-sink
   service_account_id: ajejeis235ma********
   s3:
     bucket: test-bucket
   ```

- API

   To get detailed information about a sink, use the [get](../api-ref/Sink/get.md) REST API method for the [Sink](../api-ref/Sink/index.md) resource or the [SinkService/Get](../api-ref/grpc/sink_service.md#Get) gRPC API call.

{% endlist %}