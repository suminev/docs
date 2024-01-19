# Удалить капчу


В этом разделе вы узнаете, как удалить капчу.

{% list tabs group=instructions %}

- Консоль управления {#console}

    1. В [консоли управления]({{ link-console-main }}) выберите каталог.
    1. Выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_smartcaptcha_ru }}**.
    1. Выберите капчу и нажмите ![image](../../_assets/console-icons/ellipsis.svg) → **{{ ui-key.yacloud.common.delete }}**.
    1. Подтвердите удаление.

{% endlist %}

Ключи удаленной капчи прекратят действовать. Сервис вернет сообщения об ошибке в ответ на все запросы с этими ключами.
