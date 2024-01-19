# Создание L7-балансировщика {{ alb-name }} с профилем безопасности {{ sws-name }}

C помощью сервиса {{ sws-full-name }} вы можете защитить инфраструктуру от [DDoS-атак](../glossary/ddos.md) и ботов на уровне приложений (L7).

{{ sws-name }} находится на [стадии Preview](../overview/concepts/launch-stages.md).

{% include [ask-for-turning-on](../_includes/smartwebsecurity/ask-for-turning-on.md) %}

Вы создадите тестовый веб-сервер, развернете [L7-балансировщик](../application-load-balancer/concepts/application-load-balancer.md) {{ alb-name }}, который будет распределять трафик на тестовый веб-сервер, и защитите созданную инфраструктуру с помощью [профиля безопасности](../smartwebsecurity/concepts/profiles.md) {{ sws-name }}.

Чтобы создать L7-балансировщик с профилем безопасности:
1. [Подготовьте облако к работе](#before-you-begin).
1. [Подготовьте инфраструктуру](#infrastructure-prepare).
1. [Создайте профиль безопасности](#profile-create).
1. [Подключите профиль безопасности к виртуальному хосту](#profile-connect).
1. [Протестируйте работу профиля безопасности](#test).

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).

## Подготовьте облако к работе {#before-you-begin}

{% include [before-you-begin](./_tutorials_includes/before-you-begin.md) %}

## Подготовьте инфраструктуру {#infrastructure-prepare}

[Разверните](../application-load-balancer/quickstart.md) инфраструктуру {{ alb-name }}, а также ВМ с тестовым веб-сервером.

Будут созданы следующие ресурсы:
* [ВМ](../compute/concepts/vm.md) `test-vm1` с тестовым веб-сервером.
* [целевая группа](../application-load-balancer/concepts/target-group.md) `test-target-group`.
* [группа бэкендов](../application-load-balancer/concepts/backend-group.md) `test-backend-group`.
* [HTTP-роутер](../application-load-balancer/concepts/http-router.md) `test-http-router` с [виртуальным хостом](../application-load-balancer/concepts/http-router.md#virtual-host) `test-virtual-host`.
* [L7-балансировщик](../application-load-balancer/concepts/application-load-balancer.md) `test-load-balancer`.

Сохраните публичный IP-адрес L7-балансировщика, он понадобится для проверки профиля безопасности.

{% include [auto-scaling-tip](../_includes/smartwebsecurity/auto-scaling-tip.md) %}

## Создайте профиль безопасности {#profile-create}

Профиль безопасности является основным элементом сервиса {{ sws-name }} и состоит из набора правил, каждое из которых содержит условия для фильтрации входящих пользовательских запросов к защищаемому ресурсу.

Чтобы создать профиль безопасности:

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы хотите создать профиль.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_smartwebsecurity }}**.
  1. Нажмите кнопку **{{ ui-key.yacloud.smart-web-security.action_empty }}** и выберите **{{ ui-key.yacloud.smart-web-security.title_default-template }}**.

      Преднастроенный профиль содержит:
      * [базовое правило](../smartwebsecurity/concepts/rules.md#base-rules) по умолчанию, включенное для всего трафика с [типом действия](../smartwebsecurity/concepts/rules.md#rule-action) — `{{ ui-key.yacloud.smart-web-security.overview.cell_sec-action-deny }}`;
      * [правило Smart Protection](../smartwebsecurity/concepts/rules.md#smart-protection-rules), включенное для всего трафика, с типом действия — `{{ ui-key.yacloud.smart-web-security.overview.cell_mode-full }}`.

      {% include [smart-protection-tip](../_includes/smartwebsecurity/smart-protection-tip.md) %}

  1. Введите имя профиля, например `test-sp1`.
  1. В поле **{{ ui-key.yacloud.smart-web-security.form.label_default-action }}** выберите `{{ ui-key.yacloud.smart-web-security.form.label_action-deny }}`. Таким образом, если не будет задано других правил, весь трафик к защищаемому ресурсу будет запрещен.
  1. Нажмите кнопку ![plus-sign](../_assets/console-icons/plus.svg) **{{ ui-key.yacloud.smart-web-security.form.button_add-rule }}**.
  1. В окне создания правила:
      1. Введите имя правила, например `test-rule1`.
      1. Задайте приоритет правила, например `999800`. Правило будет иметь больший приоритет, чем преднастроенные.

          {% include [preconfigured-rules-priority](../_includes/smartwebsecurity/preconfigured-rules-priority.md) %}

      1. Выберите тип правила — `{{ ui-key.yacloud.smart-web-security.overview.label_base-rule }}`.
      1. Выберите действие — `{{ ui-key.yacloud.smart-web-security.overview.cell_sec-action-allow }}`.

          В правиле будут описаны условия, при которых запросы будут направлены к бэкенду тестового приложения.
      1. В поле **{{ ui-key.yacloud.smart-web-security.overview.column_rule-conditions }}** выберите {{ ui-key.yacloud.smart-web-security.overview.condition_name-ip-range }}.
      1. В появившемся поле **Условия на IP** выберите опцию `Совпадает или принадлежит диапазону` и задайте публичный IP-адрес устройства, с которого вы будете отправлять запросы L7-балансировщику, например `158.160.100.200`.
      1. Нажмите кнопку **{{ ui-key.yacloud.common.add }}**.
        
          В блоке **{{ ui-key.yacloud.smart-web-security.form.section_security-rules }}** в таблице появится созданное вами правило.
  1. Нажмите кнопку **{{ ui-key.yacloud.common.create }}**.

{% endlist %}

## Подключите профиль безопасности к виртуальному хосту {#profile-connect}

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором вы хотите подключить профиль безопасности к виртуальному хосту сервиса {{ alb-name }}.
  1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_smartwebsecurity }}**.
  1. Выберите профиль `test-sp1`.
  1. Нажмите кнопку ![plug](../_assets/console-icons/plug-connection.svg) **{{ ui-key.yacloud.smart-web-security.overview.action_attach-to-host }}**.
  1. В открывшемся окне выберите:
      * **{{ ui-key.yacloud.smart-web-security.attach-dialog.label_balancer }}** `test-load-balancer`.
      * **{{ ui-key.yacloud.smart-web-security.attach-dialog.label_http-router }}** `test-http-router`.
      * **{{ ui-key.yacloud.smart-web-security.attach-dialog.label_virtual-host }}** `test-virtual-host`. 
  1. Нажмите кнопку **{{ ui-key.yacloud.smart-web-security.attach-dialog.action_connect }}**.

      На вкладке **{{ ui-key.yacloud.smart-web-security.overview.title_connected-to-the-hosts }}** появится подключенный виртуальный хост.

{% endlist %}

## Протестируйте работу профиля безопасности {#test}

1. Откройте терминал устройства, IP-адрес которого вы указали в разрешающем правиле.
1. Отправьте запрос на бэкенд тестового приложения:

    ```bash
    curl -v <публичный_IP-адрес_L7-балансировщика>
    ```

    В результате должно быть выведено содержимое директории с тестовым веб-сервером.

1. Повторите запрос с другого IP-адреса. В результате должно быть выведено сообщение о невозможности установить соединение с сервером.

{% note info %}

Проверка правил Smart Protection не проводится. При такой проверке параметры подозрительных запросов, в том числе IP-адреса, заносятся в черный список.

{% endnote %}

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за созданные ресурсы:
1. [Удалите](../smartwebsecurity/operations/profile-delete.md) профиль безопасности.
1. [Удалите](../application-load-balancer/operations/application-load-balancer-delete.md) L7-балансировщик.
1. [Удалите](../application-load-balancer/operations/http-router-delete.md) HTTP-роутер.
1. [Удалите](../application-load-balancer/operations/backend-group-delete.md) группу бэкендов.
1. [Удалите](../application-load-balancer/operations/target-group-delete.md) целевую группу.
1. [Удалите](../compute/operations/vm-control/vm-delete.md) ВМ.
