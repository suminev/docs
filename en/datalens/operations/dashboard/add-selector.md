# Adding a selector to a dashboard


Before adding a selector, make sure that you have the `{{ permission-write }}` or `{{ permission-admin }}` permission for a dashboard. For more information, see [this section](../../security/index.md).


To add a [selector](../../dashboard/selector.md) to a dashboard:


{% include [datalens-workbooks-collections-select-note](../../../_includes/datalens/operations/datalens-workbooks-collections-select-note.md) %}


1. In the left-hand panel, click ![image](../../../_assets/console-icons/layout-cells-large.svg) **Dashboards** and select the dashboard you need.
1. At the top of the page, click **Edit**.
1. In the panel at the bottom of the page, choose **Selector**.

   ![image](../../../_assets/datalens/operations/widget-select.png =500x118)

1. Choose your selector type:

   {% list tabs %}

   - Based on dataset

      Specify the selector parameters:

      * Under **General settings**:

         * **Dataset**: Dataset with data for the selector.
         * **Field**: Dataset field with selector values. It can be a dimension or a measure (for more information, see [{#T}](../../concepts/dataset/data-model.md#field)).
         * **Selector type**: Drop-down list, input field, calendar, or checkbox.

            {% note info %}

            * **Calendar** is only available for fields of the `Date` or `Date and time` type.
            * **Checkbox** is only available for fields of the `Boolean` type.
            * Measure selectors can only have the **Input field** type.

            {% endnote %}

         * **Operation**: Comparison operation by which the selector filters the chart values (e.g., **Equal to**, **Greater than**, or **Less than**). If the field is left blank, the selector will filter by the **Equal to** operation by default. The list of available operations depends on the field type. Do not specify an operation if the selector filters the QL chart.
         * **Multiple choice**: Allows choosing several values in the selector. Only available for the **List** selector type.
         * **Range**: Allows setting a time interval in the selector. Only available for the **Calendar** selector type.
         * **Default value**: Displayed when you first open the dashboard.

      * Under **Appearance**:

         * **Title**: Used to choose a selector when establishing a link with other widgets. The option enables you to control the display of the name on the dashboard.
         * **Inner title**: Text that is displayed in the selector to indicate a comparison operation. You can change the default value to your own one. For example, you can specify the `=` or `equal to` value for the **Equal to** operation. The parameter is only available for the **List** type.

            ![image](../../../_assets/datalens/selector-settings/selector-operation-title.png)

   - Manual input

      Specify the selector parameters:

      * Under **General settings**:

         * **Field or parameter name**: Field name, which can be used to link the selector with other widgets in the [alias](../../dashboard/link.md#alias) configuration window.

            ![image](../../../_assets/datalens/selector-settings/field-name.png)

         * **Selector type**: Drop-down list, input field, calendar, or checkbox.

            {% note info %}

            * **Calendar** is only available for fields of the `Date` or `Date and time` type.
            * **Checkbox** is only available for fields of the `Boolean` type.
            * Measure selectors can only have the **Input field** type.

            {% endnote %}

         * **Operation**: Comparison operation by which the selector filters the chart values (e.g., **Equal to**, **Greater than**, or **Less than**). If the field is left blank, the selector will filter by the **Equal to** operation by default. The list of available operations depends on the field type. Do not specify an operation if the selector filters the QL chart.
         * **Multiple choice**: Allows choosing several values in the selector. Only available for the **List** selector type.
         * **Possible values**: List of values to select. Only available for the **List** selector type.
         * **Range**: Allows setting a time interval in the selector. Only available for the **Calendar** selector type.
         * **Time**: Allows specifying time. Only available for the **Calendar** selector type.
         * **Default value**: Displayed when you first open the dashboard. This field must be set for the **List** type; otherwise, no value will be available in the selector.

      * Under **Appearance**:

         * **Title**: Used to choose a selector when establishing a link with other widgets.

            ![image](../../../_assets/datalens/selector-settings/caption.png)

            The option enables you to control the display of the name on the dashboard.

         * **Inner title**: Text that is displayed in the selector to indicate a comparison operation. You can change the default value to your own one. For example, you can specify the `=` or `equal to` value for the **Equal to** operation. The parameter is only available for the **List** type.

            ![image](../../../_assets/datalens/selector-settings/selector-operation-title.png)


   {% endlist %}

   In [QL charts](../../concepts/chart/ql-charts.md), you can control [selector parameters](../chart/create-sql-chart.md#selector-parameters) from the **Parameters** tab in the chart editing area and use the **Query** tab to specify a variable in the query itself in `not_var{{variable}}` format.

1. Click **Add**. The widget will be displayed on the dashboard.

{% note warning %}

When adding a selector by a certain field to your dashboard, filters added at the chart level are no longer applied to the dashboard's chart.

{% endnote %}

## Limitations {#restrictions}

* For measure selectors, only one selector type is available: **Input field**.
* We recommend making measure selectors independent of other selectors (you should specify the **Ignore** [link](../../dashboard/link.md) type with other selectors in the **Links** section when editing a dashboard).
