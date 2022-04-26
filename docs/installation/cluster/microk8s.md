# MicroK8s
This is the documentation for a cluster instance of the Assemblyline platform suited for larger deployments (minimum 3 compute nodes).

!!! warning "Take the following into consideration while following the relevant  instructions from the [Kubernetes appliance](../appliance/kubernetes-microk8s)."
	The changes proposed alter the storage class used in a multi-node cluster setup using OpenEBS instead of the default hostpath-based storage class.

## Setup Requirements

=== "Online"
    ```bash
    sudo microk8s enable openebs
    ```

=== "Offline"
	1. Pull the OpenEBS dependencies (internet-connected system)
	```bash
	#OpenEBS
	REPO="openebs"
	REPO_K8S="k8s.gcr.io/sig-storage"

	# Pull extra OpenEBS images
	for openebs_image in "jiva:3.1.0" "m-exporter:3.1.0" "jiva-operator:3.1.0" "jiva-csi:3.1.0"
	do
		docker pull $REPO/$openebs_image && docker save $REPO/$openebs_image >> $REPO_$openebs_image.tar
	done

	# Pull jiva addon images
	for k8s_image in "csi-attacher:v3.1.0" "livenessprobe:v2.3.0" "csi-provisioner:v3.0.0" "csi-resizer:v1.2.0" "csi-node-driver-registrar:v2.3.0"
	do
		docker pull $REPO_K8S/$k8s_image && docker save $REPO_K8S/$k8s_image >> $k8s_image.tar
	done

	# Pull cleaner Bitnami image
	docker pull bitnami/kubectl && docker save bitnami/kubectl >> bitnami_kubectl.tar

	# Download package and dependencies of iscsi, needed for OpenEBS
	apt-get -y install --print-uris open-iscsi | cut -d\' -f2 | grep http:// > open-iscsi.txt
	wget -i ./open-iscsi.txt
	```

	2. Install debian packages for open-iscsi
	```bash
	dpkg -i *.deb
	systemctl enable --now iscsid
	```

	3. Install OpenEBS manually via helm on the master node in MicroK8s
	```bash
	# Install OpenEBS helm chart
	COMMON_PATH="/var/snap/microk8s/common/"
	microk8s helm install openebs ./openebs-*.tgz --namespace openebs --create-namespace \
    --version 3.0.x \
    --set jiva.enabled=true \
    --set legacy.enabled=false \
    --set jiva.csiNode.kubeletDir="$COMMON_PATH/var/lib/kubelet/" \
    --set localprovisioner.basePath="$COMMON_PATH/var/openebs/local" \
    --set ndm.sparse.path="$COMMON_PATH/var/openebs/sparse" \
    --set varDirectoryPath.baseDir="$COMMON_PATH/var/openebs"
	```

??? note "Adding more nodes"
    **Note: This can be done before or after the system is live.**

    Install required addon:
    ```
    sudo microk8s enable ha-cluster
    ```

    From the master node, run:
    ```bash
    sudo microk8s add-node
    ```

    This will generate a command with a token to be executed on a standby node.

    On your standby node, ensure the microk8s ```ha-cluster``` addon is enabled before
    running the command from the master to join the cluster.

    To verify the nodes are connected, run (on any node):
    ```bash
    sudo microk8s kubectl get nodes
    ```

    Repeat this process for any additional standby nodes that are to be added.

    For more details, see: [Clustering with MicroK8s](https://microk8s.io/docs/clustering)


## Relevant Changes to Helm Chart
The main difference between the cluster and the appliance in MicroK8s is the use of the openEBS storage class for an Elasticsearch cluster.

The following changes will have to made to the helm chart.
```yaml
redisStorageClass: openebs-hostpath
persistantStorageClass: openebs-hostpath
...

datastore:
  replicas:  3
  volumeClaimTemplate:
    storageClassName: openebs-jiva-csi-default
...
filestore:
  persistence:
    StorageClass: openebs-hostpath
```
