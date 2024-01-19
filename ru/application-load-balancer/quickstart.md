# Как начать работать с {{ alb-full-name }}

С помощью этой инструкции вы создадите первый L7-балансировщик, подключите к нему целевую группу и настроите передачу трафика на бэкенд тестового приложения.

## Перед началом работы {#before-begin}

1. Войдите в [консоль управления]({{ link-console-main }}) или зарегистрируйтесь. Если вы еще не зарегистрированы, перейдите в консоль управления и следуйте инструкциям.
1. На странице [**{{ ui-key.yacloud.component.navigation-menu.label_billing }}**]({{ link-console-billing }}) убедитесь, что у вас подключен [платежный аккаунт](../billing/concepts/billing-account.md) и он находится в статусе `ACTIVE` или `TRIAL_ACTIVE`. Если платежного аккаунта нет, [создайте его](../billing/quickstart/index.md#create_billing_account).
1. Если у вас еще нет каталога, [создайте его](../resource-manager/operations/folder/create.md). Во время создания каталога вы можете создать виртуальную сеть по умолчанию с подсетями во всех зонах доступности.

## Создайте ВМ и запустите на ней тестовый веб-сервер {#create-vm}

1. [Создайте](../compute/operations/vm-create/create-linux-vm.md) виртуальную машину `test-vm1` в зоне доступности `{{ region-id }}-a`.
1. [Подключитесь к ВМ](../compute/operations/vm-connect/ssh.md) и запустите на ней тестовый веб-сервер, который будет отвечать на запросы на порте `80`:

    ```bash
    sudo python3 -m http.server 80
    ```

1. Убедитесь, что веб-сервер возвращает список папок из каталога. В терминале вашего компьютера выполните:

    ```bash
    curl -v <публичный_IP-адрес_тестовой_ВМ>
    ```

## Создайте целевую группу {#create-target-group}

На ВМ из [целевой группы](concepts/target-group.md) развертываются бэкенды вашего приложения. Целевая группа будет подключена к балансировщику, чтобы на эндпоинты бэкендов вашего приложения можно было направлять запросы.

Для примера в целевой группе будет находиться только одна виртуальная машина.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором будет создаваться целевая группа.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_application-load-balancer }}**.
  1. В меню слева выберите **{{ ui-key.yacloud.alb.label_target-groups }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.alb.button_target-group-create }}**.
  1. Введите имя целевой группы: `test-target-group`.
  1. Выберите ВМ `test-vm1`.
  1. Нажмите кнопку **{{ ui-key.yacloud.common.create }}**.

- CLI {#cli}

  {% include [cli-install](../_includes/cli-install.md) %}

  Выполните команду:

  ```bash
  yc alb target-group create test-target-group \
    --target subnet-name=<имя_подсети_ВМ>,ip-address=<внутренний_IP-адрес_ВМ>
  ```

{% endlist %}

## Создайте группу бэкендов {#create-backend-group}

[Группы бэкендов](concepts/backend-group.md) содержат настройки балансировки трафика и проверок состояния целевых ресурсов. Создайте группу и добавьте в нее один бэкенд.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В меню слева выберите **{{ ui-key.yacloud.alb.label_backend-groups }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.alb.button_backend-group-create }}**.
  1. Введите имя группы бэкендов: `test-backend-group`.
  1. В блоке **{{ ui-key.yacloud.alb.label_backends }}** нажмите кнопку **{{ ui-key.yacloud.common.add }}**. Задайте настройки бэкенда:
      1. Введите имя бэкенда: `backend-1`.
      1. В списке **{{ ui-key.yacloud.alb.label_target-group }}** выберите `test-target-group`.
      1. Укажите **{{ ui-key.yacloud.alb.label_port }}**: `80`.
  1. Разверните поле **{{ ui-key.yacloud.alb.label_protocol-settings }}** и задайте их параметры:
      1. Выберите тип `{{ ui-key.yacloud.alb.label_proto-http-plain }}`.
  1. Нажмите кнопку **{{ ui-key.yacloud.alb.button_add-healthcheck }}** и задайте настройки проверки:
      1. **{{ ui-key.yacloud.alb.label_timeout }}**: `1`.
      1. **{{ ui-key.yacloud.alb.label_interval }}**: `3`.
      1. **{{ ui-key.yacloud.alb.label_healthy }}**: `2`.
      1. **{{ ui-key.yacloud.alb.label_unhealthy }}**: `2`.
      1. **{{ ui-key.yacloud.common.type }}**: `{{ ui-key.yacloud.alb.label_hc-type-http }}`.
      1. **{{ ui-key.yacloud.alb.label_path }}**: `/`.
  1. Нажмите кнопку **{{ ui-key.yacloud.common.create }}**.

- CLI {#cli}

  1. Создайте группу бэкендов:

      ```bash
      yc alb backend-group create test-backend-group
      ```

  1. Создайте бэкенд и проверку состояния:

      ```bash
      yc alb backend-group add-http-backend \
        --backend-group-name test-backend-group \
        --name backend-1 \
        --port 80 \
        --target-group-name test-target-group \
        --target-group-id <идентификатор_целевой_группы> \
        --http-healthcheck healthy-threshold=2,unhealthy-threshold=2,timeout=1s,interval=3s,path=/
      ```

{% endlist %}

## Создайте HTTP-роутер {#create-http-router}

[HTTP-роутеры](concepts/http-router.md) определяют правила маршрутизации запросов к бэкендам и позволяют модифицировать запросы прямо на балансировщике.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В меню слева выберите **{{ ui-key.yacloud.alb.label_http-routers }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.alb.button_http-router-create }}**.
  1. Введите имя роутера: `test-http-router`.
  1. В блоке **{{ ui-key.yacloud.alb.label_virtual-hosts }}** нажмите кнопку **{{ ui-key.yacloud.alb.button_virtual-host-add }}**.
  1. Введите имя хоста: `test-virtual-host`.
  1. Нажмите кнопку **{{ ui-key.yacloud.alb.button_add-route }}**.
  1. Введите **{{ ui-key.yacloud.common.name }}**: `test-route`.
  1. В поле **{{ ui-key.yacloud.alb.label_path }}** выберите `{{ ui-key.yacloud.alb.label_match-prefix }}` и укажите путь `/`.
  1. В поле **{{ ui-key.yacloud.alb.label_route-action }}** оставьте `{{ ui-key.yacloud.alb.label_route-action-route }}`.
  1. В списке **{{ ui-key.yacloud.alb.label_backend-group }}** выберите `test-backend-group`.
  1. Остальные настройки оставьте без изменений и нажмите кнопку **{{ ui-key.yacloud.common.create }}**.

- CLI {#cli}

  1. Создайте HTTP-роутер:

      ```bash
      yc alb http-router create test-http-router
      ```

  1. Создайте виртуальный хост:

      ```bash
      yc alb virtual-host create test-virtual-host --http-router-name test-http-router
      ```
  
  1. Добавьте маршрут:

      ```bash
      yc alb virtual-host append-http-route test-route \
        --http-router-name test-http-router \
        --prefix-path-match / \
        --backend-group-name test-backend-group \
        --virtual-host-name test-virtual-host
      ```

{% endlist %}

## Создайте L7-балансировщик {#create-load-balancer}

[Балансировщик](concepts/application-load-balancer.md) принимает запросы и распределяет их по ВМ из целевой группы по правилам, указанным в HTTP-роутере. Для приема трафика балансировщик использует [обработчики](concepts/application-load-balancer.md#listener).

Для примера будет создан балансировщик с узлом в одной подсети в одной зоне доступности.

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В меню слева выберите **{{ ui-key.yacloud.alb.label_load-balancers }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.alb.button_load-balancer-create }}**.
  1. Введите имя балансировщика: `test-load-balancer`.
  1. В блоке **{{ ui-key.yacloud.mdb.forms.section_network-settings }}** выберите сеть, в подсетях которой будет размещаться узлы балансировщика.
  1. В блоке **{{ ui-key.yacloud.alb.section_allocation-settings }}** выберите подсети для узлов балансировщика в каждой зоне доступности и включите передачу трафика.

      {% include [subnets-required](../_includes/application-load-balancer/subnets-required.md) %}

  1. В блоке **{{ ui-key.yacloud.alb.label_listeners }}** нажмите кнопку **{{ ui-key.yacloud.alb.button_add-listener }}**. Задайте настройки обработчика:
      1. Введите имя обработчика: `test-listener`.
      1. В блоке **{{ ui-key.yacloud.alb.section_external-address-specs }}** включите передачу трафика.
      1. Укажите порт `80`.
      1. Выберите тип **{{ ui-key.yacloud.alb.label_address-auto }}**.
  1. В поле **{{ ui-key.yacloud.alb.label_http-router }}** выберите `test-http-router`.
  1. Нажмите кнопку **{{ ui-key.yacloud.common.create }}**.

- CLI {#cli}

  1. Создайте балансировщик с узлом в одной подсети:

      ```bash
      yc alb load-balancer create test-load-balancer \
        --network-name <имя_сети> \
        --location subnet-name=<имя_подсети_в_зоне_{{ region-id }}-a>,zone={{ region-id }}-a \
        --location subnet-name=<имя_подсети_в_зоне_{{ region-id }}-b>,zone={{ region-id }}-b \
        --location subnet-name=<имя_подсети_в_зоне_{{ region-id }}-c>,zone={{ region-id }}-c
      ```

  1. Добавьте обработчик:

      ```bash
      yc alb load-balancer add-listener test-load-balancer \
        --listener-name test-listener \
        --http-router-id <идентификатор_HTTP-роутера> \
        --external-ipv4-endpoint port=80
      ```

{% endlist %}

## Протестируйте работу балансировщика {#test}

В терминале выполните следующую команду:

```bash
curl -v <публичный_IP-адрес_балансировщика>:80
```

В ответ должен вернуться HTTP-ответ с кодом `200` и список папок из каталога тестовой ВМ в HTML-разметке.

После этого вы можете добавить другие виртуальные машины в целевую группу, создать новые бэкенды для вашего приложения и построить маршруты до их эндпоинтов.
