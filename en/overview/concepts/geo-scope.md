# Availability zones

An _availability zone_ is an infrastructure within a data center that hosts {{ yandex-cloud }}. {{ yandex-cloud }} resources are either _zonal_ (linked to an availability zone) or _global_ (not linked to any zone). Zonal resources include [virtual machines](../../compute/concepts/vm.md) and [disks](../../compute/concepts/disk.md). Global resources include [cloud networks](../../vpc/concepts/network.md) and [buckets](../../storage/concepts/bucket.md).

Each zone is isolated from hardware and software failures that may occur in other availability zones. By deploying your applications to multiple zones at the same time, you ensure their fault tolerance and significantly reduce the risk of data loss.


The {{ yandex-cloud }} platform is hosted in four Yandex data centers located in the Vladimir, Ryazan, Moscow, and Kaluga regions. You can host resources in the following availability zones:

* `{{ region-id }}-a`
* `{{ region-id }}-b`
* `{{ region-id }}-c` ([being deprecated](ru-central1-c-deprecation.md))
* `{{ region-id }}-d`



{% note info %}

When new availability zones appear or existing ones change location, {{ yandex-cloud }} notifies all its customers.

{% endnote %}