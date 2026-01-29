# General

## Pre-requisites

1.  A **Kubernetes** 1.18+ cluster that has an ingress controller. Assemblyline is known to work with the following Kubernetes providers:
    * Rancher
    * AKS (Azure)
    * EKS (Amazon)
    * GKE (Google)
2.  **kubectl** already configured for your cluster on your machine
3.  **helm** already configured for your cluster on your machine

## Installation

### 1. Get Assemblyline Helm chart ready

1. Add the assemblyline chart repository to helm.
   ```bash
   helm repo add assemblyline https://cybercentrecanada.github.io/assemblyline-helm-chart/
   ```
2. Make sure the chart repositories are up to date.
   ```bash
   helm repo update
   ```
3. Create a new directory of your choice which will hold your personal deployment configuration. We will refer to it as `deployment_directory`

### 2. Create the assemblyline namespace

When deploying an Assemblyline instance using our chart, it must be in its own namespace. For this documentation, we will use the `al` namespace.

``` shell
kubectl create namespace al
```

### 3. Setup secrets

In the `deployment_directory` you've just created, create a `secrets.yaml` file which will contain the different passwords used by Assemblyline.

???+ example "The secrets.yaml file should have the following format"

    ``` yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: assemblyline-system-passwords
    type: Opaque
    stringData:
      datastore-password:
      logging-password:
        # If this is the password for backends like azure blob storage, the password may need to be URL-encoded
        # if it includes non-alphanumeric characters
      filestore-password:
      initial-admin-password:
    ---
    # Initalizes secret with a temporary value, will be replaced by job upon helm install
    apiVersion: v1
    kind: Secret
    metadata:
      name: kibana-service-token
    stringData:
      token: ""
    ```

!!! tip
    Here is an example of [secrets.yaml](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/main/appliance/secrets.yaml) file used for appliance deployments.

When you're done setting the different passwords in your `secrets.yaml` file, upload it to your namespace:

```shell
kubectl apply -f <deployment_directory>/secrets.yaml --namespace=al
```

!!! warning
    From this point on, you will not need the `secret.yaml` file anymore. You should delete it.

### 4. Configure your deployment

In your `deployment_directory`, create a `values.yaml` file which will contain the configuration specific to your deployment.

!!! tip
    For an exhaustive view of all the possible parameters you can change the `values.yaml` you've created, refer to the [assemblyline-helm-chart/assemblyline/values.yaml](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/main/charts/assemblyline/values.yaml) file.

These are the strict minimum configuration changes you will need to do:

1.  Setup the ingress controller by changing the values of:
    * `ingressAnnotations.cert-manager.io/issuer:` (Name of the issuer in K8s. This is for cert validation)
    * `tlsSecretName` (Name of the TLS cert in k8s. This is for cert validation)
    * `configuration.ui.fqdn` (Domain name for your al instance).
2.  Setup the storage classes according to your Kubernetes cluster :
    * `redisStorageClass` (Use SSD backed managed disks)
    * `log-storage.volumeClaimTemplate.storageClassName` (Use SSD backed managed disks)
    * `datastore.volumeClaimTemplate.storageClassName` (Use SSD backed managed disks)
    * `persistentStorageClass` (Use standard file sharing disks)
3.  Decide where you want files stored, set the appropriate URI in the `configuration.filestore.*` fields. You should try to avoid using the internal filestore and use something like Azure blob store, Amazon S3...
4.  Enable/disable/configure logging features, (disabled by default).

???+ example "This is an example values.yaml file to get you started"
    ```yaml
    # 1. Setup the ingress controller
    ingressAnnotations:
      kubernetes.io/ingress.class: "nginx"
      nginx.ingress.kubernetes.io/proxy-body-size: 100M
      cert-manager.io/issuer: <CHANGE_ME>
    tlsSecretName: <CHANGE_ME>

    # 2. Setup the storage classes according to your Kubernetes cluster
    redisStorageClass: <CHANGE_ME>
    datastore:
      volumeClaimTemplate:
        storageClassName: <CHANGE_ME>
    log-storage:
      volumeClaimTemplate:
        storageClassName: <CHANGE_ME>
    persistantStorageClass: <CHANGE_ME>


    # 3. Decide where you want files stored
    internalFilestore: false
    # Un-comment and setup if internal filestore used
    #filestore:
    #  persistence:
    #    size: 500Gi
    #    StorageClass: <CHANGE_ME>


    # 4. Enable/disable/configure logging features
    enableLogging: false
    enableMetrics: false
    enableAPM: false
    internalELKStack: false
    seperateInternalELKStack: false
    loggingUsername: elastic
    loggingTLSVerify: "none"


    # Internal configuration for assemblyline components. See the assemblyline
    # administration documentation for more details.
    # https://cybercentrecanada.github.io/assemblyline4_docs/configuration/config_file/
    configuration:
      # 1. Setup the ingress controller
      submission:
        max_file_size: 104857600
      ui:
        fqdn: "localhost"

      # 3. Decide where you want files stored
      filestore:
        cache: ["s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-cache&use_ssl=False"]
        storage: ["s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-storage&use_ssl=False"]

      # 4. Enable/disable/configure logging features
      logging:
        log_level: WARNING
    ```

### 5. Deploy your current configuration

Now that you've fully configured your `values.yaml` file, you can simply deploy it via helm by referencing the default assemblyline helm chart.

```shell
helm install assemblyline assemblyline/assemblyline -f <deployment_directory>/values.yaml -n al
```

!!! warning
    After you've ran the `helm install` command, the system has a lot of setting up to do (Creating database indexes, loading service, setting up default accounts, loading signatures ...). Don't expect it to be fully operational for at least the next 15 minutes.

### 6. (Optional) Cluster management tools

You can manage your deployment in Kubernetes using `kubectl` but it's typically laborious to type the commands to monitor or debug your instance. For that reason, we have a few tools that we recommend using.

#### k9s \[Recommended\]

You can install the k9s CLI using a package manager or installing from [source](https://k9scli.io/)

<script src="https://asciinema.org/a/305944.js" id="asciicast-305944" async="true"></script>

#### FreeLens IDE

If the computer on which your microk8s deployment is installed has a desktop interface, we strongly suggest that you use an IDE like FreeLens to manage the system

##### Install FreeLens

You'll have to fetch the appropriate installation file from [FreeLens releases](https://github.com/freelensapp/freelens/releases) and use your package manager to install manually:

```bash
# Ubuntu
sudo snap install -y /path/to/FreeLens*.deb
```

If monitoring from a Windows system, Microsoft's SmartScreen will detect the file as being unrecognized and block execution. This can be resolved by checking 'Unblock' in the file's properties.

##### Configure FreeLens

After you run FreeLens for the first time, click the "Add cluster" menu/button, select the paste as text tab and paste the output of the following command:

```bash
sudo kubectl config view --raw
```

## Update your deployment

### Updating your configuration

If you wish to apply changes to your `values.yaml` file without changing the version of assemblyline installed you need to run the `helm upgrade` command with the current release version set.

You can see your current release version by running `helm list`.

```bash
helm list
```
Which produces an output similar to this.
```
NAME          NAMESPACE  REVISION  UPDATED                                 STATUS    CHART                    APP VERSION
assemblyline  al         2727      2026-01-29 11:00:08.07031363 -0500 EST  deployed  assemblyline-7.0.79-dev  4.7.0.dev79
```

The version we want is the one in the **CHART** column following the chart name. For this sample output that is 7.0.79-dev.

!!! warning
    Not the **APP VERSION** column.

The corresponding upgrade command would then be:
```bash
helm upgrade assemblyline assemblyline/assemblyline -f values.yaml -n al --version 7.0.79-dev
```
```bash
helm upgrade <name of deployment> <chart name> -f <values file to apply> -n <namespace> --version <version to pin>
```

### Applying minor updates

To apply a minor update run the upgrade command with the corresponding **chart version** for the update you want applied. The chart version is the same as the assemblyline version with the leading `4.` and `stable` or `dev` markers removed. So Assemblyline release `4.7.2.stable2` would be chart version `7.2.2`. Development chart versions will have a `-dev` suffix added after all the version numbers so Assemblyline release `4.7.2.dev2` would be installed with chart version `7.2.2-dev`

Make sure any lines setting the `release` value have been removed from your `values.yaml` file.

If you simply want to upgrade to the most recent safe release for assemblyline get your chart version as above and run the upgrade command with the major version pinned. For the sample system above the upgrade command would be:

```bash
helm upgrade assemblyline assemblyline/assemblyline -f values.yaml -n al --version "^7"
```

For the latest release of Assemblyline 4.7.XX.

### Applying major updates

[See the documentation section on that topic.](../../administration/system_management#assemblyline-major-upgrades)