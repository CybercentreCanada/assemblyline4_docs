# General

## Pre-requisites

1. A **Kubernetes** 1.18+ cluster that has an ingress controller. Assemblyline is known to work with the following Kubernetes providers:
    * Rancher
    * AKS (Azure)
    * EKS (Amazon)
    * GKE (Google)
2. **kubectl** already configured for your cluster on your machine
3. **helm** already configured for your cluster on your machine

## Installation

### 1. Get Assemblyline Helm chart ready

1. Download the latest [Assemblyline helm chart](https://github.com/CybercentreCanada/assemblyline-helm-chart/archive/refs/heads/master.zip)
2. Unzip it into a directory of your choice which we will refer to as `assemblyline-helm-chart`
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
    Here is an example of [secrets.yaml](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/appliance/secrets.yaml) file used for appliance deployments.

When you're done setting the different passwords in your `secrets.yaml` file, upload it to your namespace:
```shell
kubectl apply -f <deployment_directory>/secrets.yaml --namespace=al
```

!!! warning
    From this point on, you will not need the `secret.yaml` file anymore. You should delete it.

### 4. Configure your deployment

In your `deployment_directory`, create a `values.yaml` file which will contain the configuration specific to your deployment.

!!! tip
    For an exhaustive view of all the possible parameters you can change the `values.yaml` you've created, refer to the [assemblyline-helm-chart/assemblyline/values.yaml](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml) file.


These are the strict minimum configuration changes you will need to do:

1. Setup the ingress controller by changing the values of:
    * `ingressAnnotations.cert-manager.io/issuer:` (Name of the issuer in K8s. This is for cert validation)
    * `tlsSecretName` (Name of the TLS cert in k8s. This is for cert validation)
    * `configuration.ui.fqdn` (Domain name for your al instance).
2. Setup the storage classes according to your Kubernetes cluster :
    * `redisStorageClass` (Use SSD backed managed disks)
    * `log-storage.volumeClaimTemplate.storageClassName` (Use SSD backed managed disks)
    * `datastore.volumeClaimTemplate.storageClassName` (Use SSD backed managed disks)
    * `persistentStorageClass` (Use standard file sharing disks)
3. Decide where you want files stored, set the appropriate URI in the `configuration.filestore.*` fields. You should try to avoid using the internal filestore and use something like Azure blob store, Amazon S3...
4. Enable/disable/configure logging features, (disabled by default).

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
helm install assemblyline <assemblyline-helm-chart>/assemblyline -f <deployment_directory>/values.yaml -n al
```

!!! warning
    After you've ran the `helm install` command, the system has a lot of setting up to do (Creating database indexes, loading service, setting up default accounts, loading signatures ...). Don't expect it to be fully operational for at least the next 15 minutes.

## Update your deployment

Once you have your Assemblyline chart deployed through helm, you can change any values in the `values.yaml` file and upgrade your deployment with the following command:

```shell
helm upgrade assemblyline <assemblyline-helm-chart>/assemblyline -f <deployment_directory>/values.yaml -n al
```
