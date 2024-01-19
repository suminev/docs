To create an API gateway:

1. Describe the parameters of the `yandex_api_gateway` resource in the configuration file:

   * `name`: API gateway name. The name format is as follows:

      {% include [name-format](../../_includes/name-format.md) %}

   * `description`: API gateway description.
   * `labels`: [Labels](../../resource-manager/concepts/labels.md) for the API gateway. Set a key-value pair.
   * `spec`: API gateway specification.

   Here is an example of the configuration file structure:

   ```hcl
   resource "yandex_api_gateway" "test-api-gateway" {
     name        = "<API_gateway_name>"
     description = "<API_gateway_description>"
     labels      = {
       label       = "label"
       empty-label = ""
     }
     spec = <<-EOT
       openapi: "3.0.0"
       info:
         version: 1.0.0
         title: Test API
       paths:
         /hello:
           get:
             summary: Say hello
             operationId: hello
             parameters:
               - name: user
                 in: query
                 description: User name to appear in greetings
                 required: false
                 schema:
                   type: string
                   default: 'world'
             responses:
               '200':
                 description: Greeting
                 content:
                   'text/plain':
                     schema:
                       type: "string"
             x-yc-apigateway-integration:
               type: dummy
               http_code: 200
               http_headers:
                 'Content-Type': "text/plain"
               content:
                 'text/plain': "Hello, {user}!\n"
     EOT
   }
   ```

   For more information about resource parameters in {{ TF }}, see the [provider documentation]({{ tf-provider-resources-link }}/api_gateway).

1. Make sure the configuration files are valid.

   1. In the command line, go to the directory where you created the configuration file.
   1. Run a check using this command:

      ```
      terraform plan
      ```

   If the configuration is described correctly, the terminal will display a list of created resources and their parameters. If the configuration contains any errors, {{ TF }} will point them out.

1. Deploy cloud resources.

   1. If the configuration does not contain any errors, run this command:

      ```
      terraform apply
      ```

   1. Confirm creating the resources: type `yes` in the terminal and press **Enter**.

      All the resources you need will then be created in the specified folder. You can check the new resources and their configuration using the [management console]({{ link-console-main }}) or this [CLI](../../cli/quickstart.md) command:

      ```
      yc serverless api-gateway get <API_gateway_name>
      ```
