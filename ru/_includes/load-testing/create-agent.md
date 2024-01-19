1. Если у вас еще нет пары [SSH-ключей](../../glossary/ssh-keygen.md), [создайте их](../../compute/operations/vm-connect/ssh.md#creating-ssh-keys).
1. Создайте агент:

   {% list tabs group=instructions %}

   - Консоль управления {#console}

     1. В [консоли управления]({{ link-console-main }}) выберите [каталог](../../resource-manager/concepts/resources-hierarchy.md#folder), в котором будет создан [агент](../../load-testing/concepts/agent.md).
     1. В списке сервисов выберите **{{ ui-key.yacloud.iam.folder.dashboard.label_load-testing }}**.
     1. На вкладке **{{ ui-key.yacloud.load-testing.label_agents-list }}** нажмите кнопку **{{ ui-key.yacloud.load-testing.button_create-agent }}**.
     1. Укажите имя агента, например `agent-008`.
     1. Укажите ту же [зону доступности](../../overview/concepts/geo-scope.md), в которой находится цель тестирования.
     1. В блоке **{{ ui-key.yacloud.load-testing.section_agent }}**:
        * Выберите подходящий тип агента. Подробнее см. в разделе [Производительность агентов](../../load-testing/concepts/agent.md#benchmark).
        * Укажите [подсеть](../../vpc/concepts/network.md#subnet), в которой находится цель тестирования. В подсети должен быть создан и [настроен NAT-шлюз](../../vpc/operations/create-nat-gateway.md).
        * Если вам доступны [группы безопасности](../../vpc/concepts/security-groups.md), выберите заранее настроенную группу безопасности агента.
     1. В блоке **{{ ui-key.yacloud.compute.instances.create.section_access }}** укажите данные для доступа к агенту:
        * Выберите [сервисный аккаунт](../../iam/concepts/users/service-accounts.md) `sa-loadtest`.
        * В поле **{{ ui-key.yacloud.compute.instances.create.field_user }}** введите имя пользователя.

          {% note alert %}

          Не используйте логин `root` или другие имена, зарезервированные операционной системой. Для выполнения операций, требующих прав суперпользователя, используйте команду `sudo`.

          {% endnote %}
              
        * В поле **{{ ui-key.yacloud.compute.instances.create.field_key }}** вставьте содержимое файла [открытого ключа](../../compute/operations/vm-connect/ssh.md#creating-ssh-keys).
     1. Нажмите **{{ ui-key.yacloud.common.create }}**.
     1. Дождитесь завершения процесса создания [виртуальной машины](../../compute/concepts/vm.md). Статус агента должен смениться на `Ready for test`.

        {% note info %}

        Процесс создания агента может остановиться на статусе `Initializing connection`, если не выполнены условия:
        * У агента есть [доступ](../../load-testing/operations/security-groups-agent.md) к `loadtesting.{{ api-host }}:443` и [публичный IP-адрес](../../vpc/concepts/address.md#public-addresses).
        * В целевой подсети настроен NAT-шлюз.
        * У сервисного аккаунта, который назначен агенту, есть необходимые [роли](../../load-testing/operations/create-agent.md#infrastructure-prepare).

        {% endnote %}

   {% endlist %}

1. Привяжите публичный IP-адрес к агенту для доступа по протоколу SSH:

   {% list tabs group=instructions %}

   - Консоль управления {#console}

     1. В [консоли управления]({{ link-console-main }}) выберите каталог, в котором размещен агент.
     1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_compute }}**.
     1. Выберите ВМ с именем `agent-008`.
     1. В блоке **{{ ui-key.yacloud.compute.instance.overview.label_network-interface }}** в правом верхнем углу нажмите ![image](../../_assets/horizontal-ellipsis.svg) и выберите **{{ ui-key.yacloud.compute.instance.overview.button_add-public-ip }}**.
     1. В открывшемся окне:
        * В поле **{{ ui-key.yacloud.component.compute.one-to-one-nat-form.field_external-type }}** выберите получение адреса **{{ ui-key.yacloud.component.compute.one-to-one-nat-form.switch_auto }}**.
        * Нажмите **{{ ui-key.yacloud.component.compute.one-to-one-nat-form.button_submit }}**.

   {% endlist %}