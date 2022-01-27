# Appliance installation (MicroK8s)

This is the documentation for an appliance instance of the Assemblyline platform suited for smaller-scale deployments. Since we've used microk8s as the backend for this, the appliance setup can later be scaled to multiple nodes.

## Setup requirements

!!! info "Caveat"
    The documentation provided here assumes that you are installing your appliance on an Ubuntu-based system and was only tested on Ubuntu 20.04. You might have to change the commands a bit if you use other Linux distributions.

    The recommended minimum system requirement for this appliance is **6 CPU** and **12 GB** of Ram.

### Install pre-requisites:

=== "Offline-Cluster"

#### T.O.C

==== "On an internet-connected system"

    1. Download offline packages
    2. Install Microk8s on Master-Node 
    3. Optional - Future upgrade the easy way
       *Configure microk8s registry (localhost:32000) to handle our offline packages*
       
 !!! info "Caveat"
    Without a registry (can be done with docker registry also) you would need to update new images to every node in your deployment.
    With a registry, your nodes can pull new images from 1 registry instance managed on Master-node.
    
    4. Updating AL4 helm-chart source files to support our multi-node cluster
    5. Recomendations :)
    

    

=== "Online"

    1. Install microk8s:
    ```bash
    sudo snap install microk8s --classic
    ```
    2. Install microk8s addons:
    ```bash
    sudo microk8s enable dns ha-cluster storage metrics-server
    ```
    3. Install Helm and set it up to use with microk8s:
    ```bash
    sudo snap install helm --classic
    sudo mkdir /var/snap/microk8s/current/bin
    sudo ln -s /snap/bin/helm /var/snap/microk8s/current/bin/helm
    ```
    4. Install git:
    ```bash
    sudo apt install git
    ```
    5. Install ingress controller:
    ```bash
    sudo microk8s kubectl create ns ingress
    sudo microk8s helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    sudo microk8s helm repo update
    sudo microk8s helm install ingress-nginx ingress-nginx/ingress-nginx --set controller.hostPort.enabled=true -n ingress
    ```
=== "Offline"

    1. Download offline packages (On an internet-connected system):

        Create a directory to house everything to be transferred
        ```bash
        mkdir al_deps
        cd al_deps
        ```
        Download offline snap packages
        ```bash
        for snap_pkg in "microk8s" "helm" "kubectl"
        do
            sudo snap download $snap_pkg
        done
        ```
        Fetch helm charts
        ```bash
        # Clone the Assemblyline helm chart repo
        git clone https://github.com/CybercentreCanada/assemblyline-helm-chart

        # Download dependency helm charts
        helm dependency update assemblyline-helm-chart/assemblyline/

        wget https://github.com/kubernetes/ingress-nginx/releases/download/helm-chart-4.0.12/ingress-nginx-4.0.12.tgz
        ```
        Fetch  container images
        ```bash
        # Calico CNI
        for container_image in "cni" "pod2daemon-flexvol" "node"
        do
            docker pull calico/$container_image:v3.19.1 && docker save calico/$container_image:v3.19.1 >> calico_$container_image.tar
        done
        docker pull calico/kube-controllers:v3.17.3 && docker save calico/kube-controllers:v3.17.3 >> calico_kube-controllers.tar

        # NGINX-Ingress, refer to values.yaml in `charts` directory
        for container_image in "controller:v1.1.0" "kube-webhook-certgen:v1.1.1"
        do
            docker pull k8s.gcr.io/ingress-nginx/$container_image && docker save k8s.gcr.io/ingress-nginx/$container_image >> $container_image.tar
        done

        # microk8s add-ons, refer to images used in corresponding .yaml files (https://github.com/ubuntu/microk8s/tree/master/microk8s-resources/actions)
        export ARCH=amd64

        # DNS, Storage, coreDNS, metrics-server container images
        for container_image in "k8s-dns-kube-dns" "k8s-dns-dnsmasq-nanny" "k8s-dns-sidecar"
        do
            docker pull gcr.io/google_containers/$container_image-$ARCH:1.14.7 && docker save gcr.io/google_containers/$container_image-$ARCH:1.14.7 >> $container_image.tar
        done
        docker pull k8s.gcr.io/pause:3.1 && docker save k8s.gcr.io/pause:3.1 >> pause.tar
        docker pull coredns/coredns:1.8.0 && docker save coredns/coredns:1.8.0 >> coredns.tar
        docker pull cdkbot/hostpath-provisioner-$ARCH:1.0.0 && docker save cdkbot/hostpath-provisioner-$ARCH:1.0.0 >> storage.tar
        docker pull k8s.gcr.io/metrics-server/metrics-server:v0.5.2 && docker save k8s.gcr.io/metrics-server/metrics-server:v0.5.2 >> metrics.tar

        # Assemblyline Core (release: 4.2.stable)
        for al_image in "core" "ui" "ui-frontend" "service-server" "socketio"
        do
            docker pull cccs/assemblyline-$al_image:4.2.stable && docker save cccs/assemblyline-$al_image:4.2.stable >> al_$al_image.tar
        done

        # Elastic
        ES_REG=docker.elastic.co
        ES_VER=7.15.0
        for beat in "filebeat" "metricbeat"
        do
            docker pull $ES_REG/beats/$beat:$ES_VER && docker save $ES_REG/beats/$beat:$ES_VER >> es_$beat.tar
        done
        for es in "logstash" "kibana" "elasticsearch"
        do
            docker pull $ES_REG/$es/$es:$ES_VER && docker save $ES_REG/$es/$es:$ES_VER >> es_$es.tar
        done

        # Filestore image (MinIO)
        for minio_image in "minio:RELEASE.2021-02-14T04-01-33Z" "mc:RELEASE.2021-02-14T04-28-06Z"
        do
            docker pull minio/$minio_image && docker save minio/$minio_image >> minio_$minio_image.tar
        done

        # Redis image
        docker pull redis && docker save redis >> redis.tar
        ```
        (Optional) Container Registry Image
        ```bash
        docker pull registry && docker save registry >> registry.tar
        ```
        (Optional) Pull official service images
        ```bash
        for svc_image in apkaye beaver characterize configextractor cuckoo deobfuscripter emlparser espresso extract floss frankenstrings iparse metadefender metapeek oletools pdfid peepdf pefile pixaxe safelist sigma suricata swiffer tagcheck torrentslicer unpacme unpacker vipermonkey virustotal-dynamic virustotal-static xlmmacrodeobfuscator yara
        do
            docker pull cccs/assemblyline-service-$svc_image:stable && docker save cccs/assemblyline-service-$svc_image:stable >> $svc_image.tar
        done
        ```

    2. Load images into a container registry

        (Optional) Setup local container registry
        ```bash
        # Assumes Docker is installed on hosting system
        for container_image in registry.tar):
        do
            sudo docker load -i $container_image
        done

        # Start up registry container
        sudo docker run -dp 32000:5000 --restart=always --name registry registry
        ```
        Load images from disk and push to registry
        ```bash
        REGISTRY=localhost:32000
        # Re-tag images and push to local registry
        for image in $(docker image ls --format {{.Repository}}:{{.Tag}})
        do
            image_tag=$image
            if [ $(grep -o '/' <<< $image | wc -l) -eq 2 ]
            then
                image_tag=$(cut -d '/' -f 2- <<< $image)
            fi
            sudo docker tag $image $REGISTRY/$image_tag && docker push $REGISTRY/$image_tag
            sudo docker image rm $image
            sudo docker image rm $REGISTRY/$image_tag
        done
        ```

    3. Setup computing host(s):
        1. Install microk8s
        ```bash
        sudo snap ack microk8s_*.assert
        sudo snap install microk8s_*.snap --classic
        ```

        2.  Modify Container Registry Endpoints
        ```bash
        sudo vim /var/snap/microk8s/current/args/containerd-template.toml
        ```
        Replace _REGISTRY_ with the domain/port of your container registry (ie. localhost:32000)
        ```toml
            [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
              [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
                endpoint = ["https://registry-1.docker.io", "http://<REGISTRY>", ]
              [plugins."io.containerd.grpc.v1.cri".registry.mirrors."*"]
                endpoint = ["http://<REGISTRY>"]
        ```
        Restart containerd to acknowledge the changes
        ```bash
        sudo systemctl restart containerd
        ```

        3. Enable microk8s add-ons
        ```bash
        sudo microk8s reset && sudo microk8s enable dns ha-cluster storage metrics-server
        ```
        4. Fetch kubeconfig for administration
        ```bash
        cp /var/snap/microk8s/current/credentials/client.config <remote_destination>
        ```

    4. Install admin tools (separate or same host(s) for computing):
    ```bash
    sudo snap ack helm_*.assert
    sudo snap install helm_*.snap --classic

    sudo snap ack kubectl_*.assert
    sudo snap install kubectl_*.snap --classic

    # Copy kubeconfig from cluster and make it accessible for kubectl/helm
    export KUBECONFIG=$HOME/.kube/config

    # If installing on computing hosts
    # sudo mkdir /var/snap/microk8s/current/bin
    # sudo ln -s /snap/bin/helm /var/snap/microk8s/current/bin/helm
    # sudo ln -s /snap/bin/kubectl /var/snap/microk8s/current/bin/kubectl

    # (Optional) Install additional Kubernetes monitoring tools like k9s or Lens
    ```

    5. Install ingress:
    ```bash
    # These steps assume you know the digest of the re-tagged images required for the ingress-nginx helm chart
    kubectl create ns ingress
    helm install ingress-nginx ./ingress-nginx-*.tgz --set controller.hostPort.enabled=true -n ingress
    ```

??? note "Adding more nodes (optional)"
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

## Get the Assemblyline chart to your administration computer

### Get the Assemblyline helm charts
```bash
mkdir ~/git && cd ~/git
git clone https://github.com/CybercentreCanada/assemblyline-helm-chart.git
```

### Create your personal deployment

```bash
mkdir ~/git/deployment
cp ~/git/assemblyline-helm-chart/appliance/*.yaml ~/git/deployment
```

### Setup the charts and secrets

The ```values.yaml``` file in your deployment directory ```~/git/deployment``` is already pre-configured for use with microk8s as a basic one node minimal appliance. Make sure you go through the file to adjust disk sizes and to turn on/off features to your liking.

The ```secret.yaml``` file in your deployment directory is preconfigured with default passwords, you should change them.

!!! tip
    The secrets are used to set up during bootstrap so make sure you change them before deploying the ```al``` chart.
!!! warning
    Be sure to update any values as deemed necessary ie. FQDN

## Deploy Assemblyline via Helm:

### Create a namespace for your Assemblyline install

For this documentation, we will use ```al``` as the namespace.

```bash
sudo microk8s kubectl create namespace al
```

### Deploy the secret to the namespace

```bash
sudo microk8s kubectl apply -f ~/git/deployment/secrets.yaml --namespace=al
```

From this point on, you don't need the secrets.yaml file anymore. You should delete it so there is no file on disk containing your passwords.

```bash
rm ~/git/deployment/secrets.yaml
```

### Finally, let's deploy Assemblyline's chart:

For documentation, we will use ```assemblyline``` as the deployment name.

```bash
sudo microk8s helm install assemblyline ~/git/assemblyline-helm-chart/assemblyline -f ~/git/deployment/values.yaml -n al
```

!!! warning
    After you've ran the `helm install` command, the system has a lot of setting up to do (Creating database indexes, loading service, setting up default accounts, loading signatures ...). Don't expect it to be fully operational for at least the next 15 minutes.


## Updating the current deployment

Once you have your Assemblyline chart deployed through helm, you can change any values in the ```values.yaml``` file and upgrade your deployment with the following command:
```bash
sudo microk8s helm upgrade assemblyline ~/git/assemblyline-helm-chart/assemblyline -f ~/git/deployment/values.yaml -n al
```

??? tip "(Optional) Get the latest assemblyline-helm-chart"
    Before doing your `helm upgrade` command, you can get the latest changes that we did to the chart by pulling them. (This could conflict with the changes you've made so be careful while doing this.)
    ```bash
    cd ~/git/assemblyline-helm-chart && git pull
    ```

## Quality of life improvements

### Lens IDE
If the computer on which your microk8s deployment is installed has a desktop interface, I strongly suggest that you use K8s Lens IDE to manage the system

#### Install Lens
```bash
sudo snap install kontena-lens --classic
```
#### Configure Lens
After you run Lens for the first time, click the "Add cluster" menu/button, select the paste as text tab and paste the output of the following command:
```bash
sudo microk8s kubectl config view --raw
```

### Sudoless access to MicroK8s
MicroK8s require you to add sudo in front of every command, you can add your user to the microk8s group so you don't have to.

```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```

!!! warning "You will need to reboot for these changes to take effect"
    Temporarily, you can add the group to your current shell by running the following:
    ```
    newgrp microk8s
    ```

### Alias to Kubectl
Since all is running inside microk8s you can create an alias to the kubectl command to make your life easier
```
sudo snap alias microk8s.kubectl kubectl
kubectl config set-context --current --namespace=al
```

## Alternative Installations

We will officially only support microk8s installations for appliances, but you can technically install it on any local Kubernetes frameworks (k3s, minikube, kind...). That said there will be no documentation for these setups, and you will have to modify the ```values.yaml``` storage classes to fit with your desired framework.
