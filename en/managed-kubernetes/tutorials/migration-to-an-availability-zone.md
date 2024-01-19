# Migrating {{ managed-k8s-name }} resources to a different availability zone


{% note info %}

{{ managed-k8s-name }} only supports [node group](../concepts/index.md#node-group) migration. Going forward, you will also be able to migrate your [master host](../concepts/index.md#master) to another availability zone.

{% endnote %}

## Getting started {#before-you-begin}

{% include [cli-install](../../_includes/cli-install.md) %}

{% include [default-catalogue](../../_includes/default-catalogue.md) %}


## Migrate the node group and the pod workloads to a different availability zone {#transfer-a-node-group}

Node group migration works differently depending on the type of workload in the pods:

* [Stateless workload](#stateless). The functioning of applications within the pods during migration depends on how the workload is distributed among the cluster nodes. If the pods are deployed both in a node group being migrated and groups where the availability zone stays the same, the applications continue to run. If the pods are only deployed in the migrating group, both the pods and the applications within them will have to be stopped for a short period of time.

   Examples of stateless workloads include the web server, {{ alb-full-name }} [Ingress controller](../../application-load-balancer/tools/k8s-ingress-controller/index.md), and REST API applications.

* With [stateful workloads](#stateful), the pods and the applications have to be stopped for a short period of time, regardless of how the workload is distributed among the cluster nodes.

   Examples of stateful workloads include databases and storages.

### Migrating a stateless workload {#stateless}

1. Check if the `nodeSelector`, `affinity`, or `topology spread constraints` strategies are being used to assign the pods to the group's nodes. For more information on strategies, see [{#T}](../concepts/usage-recommendations.md#high-availability) and the [{{ k8s }} documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/). To check how a pod is assigned to its associated node and unlink them:

   {% list tabs %}

   - Management console

      1. In the [management console]({{ link-console-main }}), select the folder with your {{ managed-k8s-name }} cluster.
      1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
      1. Go to the cluster page and find the **{{ ui-key.yacloud.k8s.cluster.switch_workloads }}** section.
      1. On the **{{ ui-key.yacloud.k8s.workloads.label_pods }}** tab, open the pod's page.
      1. Go to the **{{ ui-key.yacloud.k8s.workloads.label_tab-yaml }}** tab.
      1. Check if the pod manifest contains the following parameters and {{ k8s }} labels in them:

         * Parameters:

            * `spec.nodeSelector`
            * `spec.affinity`
            * `spec.topologySpreadConstraints`

         * {{ k8s }} labels set in the parameters:

            * `failure-domain.beta.kubernetes.io/zone`: <availability_zone>
            * `topology.kubernetes.io/zone`: <availability_zone>

         If a configuration has at least one of these parameters and that parameter contains at least one of the listed {{ k8s }} labels, that configuration prevents node group and workload migration.

      1. Check if the pod manifest has any dependencies from:

         * Availability zone you are migrating resources from.
         * Specific nodes within the group.

      1. If you find any of these settings, assignments, or dependencies, remove them from the pod configuration:

         1. Copy the YAML configuration from the management console.
         1. Create a local YAML file and paste the configuration into it.
         1. Remove any availability zone assignments from the configuration. For example, if the `spec.affinity` parameter sets the `failure-domain.beta.kubernetes.io/zone` {{ k8s }} label, remove it.
         1. Apply the new configuration:

            ```bash
            kubectl apply -f <path_to_YAML_file>
            ```

         1. Make sure the pod status changed to `Running`:

            ```bash
            kubectl get pods
            ```

      1. Check and update the configuration of each pod by repeating these steps.

   {% endlist %}

1. Create a subnet in the availability zone you want to move your node group to.

   {% list tabs %}

   - CLI

      ```bash
      yc vpc subnet create \
         --name <subnet_name> \
         --zone <availability_zone> \
         --network-id <network_ID> \
         --range <subnet_CIDR>
      ```

      In the command, specify the following subnet parameters:

      * `--name`: Subnet name.
      * `--zone`: Availability zone (`{{ region-id }}-a`, `{{ region-id }}-b`, or `{{ region-id }}-d`).
      * `--network-id`: ID of the network the new subnet belongs to.
      * `--range`: List of IPv4 addresses for outgoing and incoming traffic, e.g., `10.0.0.0/22` or `192.168.0.0/16`. Make sure the addresses are unique within the network. The minimum subnet size is `/28`, the maximum subnet size is `/16`. Only IPv4 is supported.

   {% endlist %}

1. Migrate your node group to a different availability zone:

   {% list tabs %}

   - CLI

      ```bash
      {{ yc-k8s }} node-group update \
         --id <node_group_ID> \
         --location zone=<availability_zone>,subnet-id=<subnet_ID> \
         --network-interface subnets=<subnet_ID>
      ```

      This command recreates the nodes within their group in the specified availability zone and subnet. When recreating, the deployment settings are considered: the maximum number of nodes by which you can increase or decrease the group size versus the original node count.

      In the command, set the following parameter values:

      * `--id`: ID of the node group to migrate to a different availability zone.
      * `zone`: New availability zone.
      * `subnet-id` and `subnets`: ID of the new subnet that you created when migrating the master host.

   {% endlist %}

1. Make sure the pods are running in the migrated node group:

   ```bash
   kubectl get po --output wide
   ```

   The output of this command displays the nodes on which your pods are currently running.

### Migrating a stateful workload {#stateful}

The migration process is based on scaling the `StatefulSet` controller. To migrate a stateful workload:

1. Get a list of `StatefulSet` controllers to find the name of the one you need:

   ```bash
   kubectl get statefulsets
   ```

1. Get the number of pods managed by the controller:

   ```bash
   kubectl get statefulsets <controller_name> \
      -n default -o=jsonpath='{.status.replicas}'
   ```

   Save the obtained value. You will need it to scale the StatefulSet controller once the migration of your stateful workload is complete.

1. Reduce the number of pods to zero:

   ```bash
   kubectl scale statefulset <controller_name> --replicas=0
   ```

   This way, you will disable the pods that use disks. In addition, running this command saves a [PersistentVolumeClaim](../concepts/volume.md#persistent-volume) (PVC) {{ k8s }} API object.

1. Create a snapshot that represents a point-in-time copy of the [PersistentVolume](../concepts/volume.md#persistent-volume) (PV). For more information about snapshots, see the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volume-snapshots/).

   1. Get the name of the `PersistentVolumeClaim`:

      ```bash
      kubectl get pvc
      ```

   1. Create a `snapshot.yaml` file with the snapshot manifest and specify the `PersistentVolumeClaim` name in it:

      ```yaml
      apiVersion: snapshot.storage.k8s.io/v1
      kind: VolumeSnapshot
      metadata:
         name: new-snapshot-test
      spec:
         volumeSnapshotClassName: yc-csi-snapclass
         source:
            persistentVolumeClaimName: <PVC_name>
      ```

   1. Create a snapshot:

      ```bash
      kubectl apply -f snapshot.yaml
      ```

   1. Make sure the snapshot has been created:

      ```bash
      kubectl get volumesnapshots.snapshot.storage.k8s.io
      ```

   1. Make sure the [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#introduction) {{ k8s }} API object has been created:

      ```bash
      kubectl get volumesnapshotcontents.snapshot.storage.k8s.io
      ```

1. Get the snapshot ID:

   ```bash
   yc compute snapshot list
   ```

1. Create a [VM disk](../../compute/concepts/disk.md) from the snapshot:

   ```bash
   yc compute disk create \
      --source-snapshot-id <snapshot_ID> \
      --zone <availability_zone>
   ```

   In the command, specify the availability zone to which the {{ managed-k8s-name }} node group is being migrated.

   Save the following parameters from the command's output:
   * Disk ID from the `id` field.
   * Disk type from the `type_id` field.
   * Disk size from the `size` field.

1. Create a `PersistentVolume` {{ k8s }} API object from the new disk:

   1. Create a `persistent-volume.yaml` file with the `PersistentVolume` manifest:

      ```yaml
      apiVersion: v1
      kind: PersistentVolume
      metadata:
         name: new-pv-test
      spec:
         capacity:
            storage: <PersistentVolume_size>
         accessModes:
            - ReadWriteOnce
         csi:
            driver: disk-csi-driver.mks.ycloud.io
            fsType: ext4
            volumeHandle: <disk_ID>
         storageClassName: <disk_type>
      ```

      In the file, specify the parameters of the disk created from the snapshot:

      * `spec.capacity.storage`: Disk size.
      * `spec.csi.volumeHandle`: Disk ID.
      * `spec.storageClassName`: Disk type. Specify the type in accordance with the following table:

         | Type of the disk created from the snapshot | Disk type for the YAML file |
         | ----------- | ----------- |
         | `network-ssd` | `yc-network-ssd` |
         | `network-ssd-nonreplicated` | `yc-network-ssd-nonreplicated` |
         | `network-nvme` | `yc-network-nvme` |
         | `network-hdd` | `yc-network-hdd` |

   1. Create a `PersistentVolume` object:

      ```bash
      kubectl apply -f persistent-volume.yaml
      ```

   1. Make sure the `PersistentVolume` has been created:

      ```bash
      kubectl get pv
      ```

      The `new-pv-test` object will appear in the command's output.

1. Create a `PersistentVolumeClaim` object from the new `PersistentVolume`:

   1. Create a `persistent-volume-claim.yaml` file with the `PersistentVolumeClaim` manifest:

      ```yaml
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
         name: <PVC_name>
      spec:
         accessModes:
            - ReadWriteOnce
         resources:
            requests:
               storage: <PV_size>
         storageClassName: <disk_type>
         volumeName: new-pv-test
      ```

      In the file, set the following parameters:

      * `metadata.name`: Name of the `PersistentVolumeClaim` that you used to create the snapshot. You can get this name by running the `kubectl get pvc` command.
      * `spec.resources.requests.storage`: Size of the `PersistentVolume`; matches the size of the created disk.
      * `spec.storageClassName`: Disk type of the `PersistentVolume`; matches the disk type of the new `PersistentVolume`.

   1. Delete the original `PersistentVolumeClaim` so you can replace it:

      ```bash
      kubectl delete pvc <PVC_name>
      ```

   1. Create a `PersistentVolumeClaim`:

      ```bash
      kubectl apply -f persistent-volume-claim.yaml
      ```

   1. Make sure the `PersistentVolumeClaim` has been created:

      ```bash
      kubectl get pvc
      ```

      The output of the `PersistentVolumeClaim` command will contain the size you specified in the YAML file.

1. Create a subnet in the availability zone you want to move your node group to.

   {% list tabs %}

   - CLI

      ```bash
      yc vpc subnet create \
         --name <subnet_name> \
         --zone <availability_zone> \
         --network-id <network_ID> \
         --range <subnet_CIDR>
      ```

      In the command, specify the following subnet parameters:

      * `--name`: Subnet name.
      * `--zone`: Availability zone (`{{ region-id }}-a`, `{{ region-id }}-b`, or `{{ region-id }}-d`).
      * `--network-id`: ID of the network the new subnet belongs to.
      * `--range`: List of IPv4 addresses for outgoing and incoming traffic, e.g., `10.0.0.0/22` or `192.168.0.0/16`. Make sure the addresses are unique within the network. The minimum subnet size is `/28`, the maximum subnet size is `/16`. Only IPv4 is supported.

   {% endlist %}

1. Migrate your node group to a different availability zone:

   {% list tabs %}

   - CLI

      ```bash
      {{ yc-k8s }} node-group update \
         --id <node_group_ID> \
         --location zone=<availability_zone>,subnet-id=<subnet_ID> \
         --network-interface subnets=<subnet_ID>
      ```

      This command recreates the nodes within their group in the specified availability zone and subnet. When recreating, the deployment settings are considered: the maximum number of nodes by which you can increase or decrease the group size versus the original node count.

      In the command, set the following parameter values:

      * `--id`: ID of the node group to migrate to a different availability zone.
      * `zone`: New availability zone.
      * `subnet-id` and `subnets`: ID of the new subnet that you created when migrating the master host.

   {% endlist %}

1. Restore the original number of pods managed by the `StatefulSet` controller:

   ```bash
   kubectl scale statefulset <controller_name> --replicas=<pod_count>
   ```

   The pods will be launched in the migrated node group.

   In the command, specify the following parameters:

   * Name of the `StatefulSet` controller. You can get it by running the `kubectl get statefulsets` command.
   * Number of pods prior to scaling down the controller.

1. Make sure the pods are running in the migrated node group:

   ```bash
   kubectl get po --output wide
   ```

   The output of this command displays the nodes on which your pods are currently running.
