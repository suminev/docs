# Отмена удаления облака

В этом разделе приведена инструкция по отмене удаления облака в случае, если ранее вы ошибочно [инициировали удаление](delete.md). Об отмене удаления облака при удалении из-за задолженности читайте в разделах [Цикл оплаты для физических лиц](../../../billing/payment/billing-cycle-individual.md) и [Цикл оплаты для организаций и ИП](../../../billing/payment/billing-cycle-business.md).

Отменить удаление можно, пока облако находится в статусе ожидания удаления `PENDING_DELETION`.


{% include [alert-pending-deletion](../../../_includes/resource-manager/alert-pending-deletion.md) %}


Чтобы отменить удаление облака, у вас должна быть роль `{{ roles-cloud-owner }}` на это облако. Если вы не можете выполнить эту операцию, обратитесь к [владельцу облака](../../concepts/resources-hierarchy.md#owner).

{% list tabs group=instructions %}

- Консоль управления {#console}

  1. В [консоли управления]({{ link-console-main }}) в списке слева выберите нужное облако.
  
  1. В правом верхнем углу нажмите значок ![***](../../../_assets/console-icons/ellipsis.svg) и выберите **{{ ui-key.yacloud.component.console-dashboard.button_cancel-deletion }}**.

  Начатая ранее операция удаления облака будет отменена.

- API {#api}

  Сейчас нет возможности отменить удаление облака с помощью API.

{% endlist %}


