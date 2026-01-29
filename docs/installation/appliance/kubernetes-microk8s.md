# Appliance in Kubernetes

!!! warning "Instructions below are tailored for an appliance setup using MicroK8s."

This is the documentation for an appliance instance of the Assemblyline platform suited for smaller-scale deployments.

## Setup requirements

!!! info "Caveat"
    The documentation provided here assumes that you are installing your appliance on an Ubuntu-based system and was only tested on Ubuntu 20.04, 22.04, and 24.04. You might have to change the commands a bit if you use other Linux distributions.

    The recommended minimum system requirement for this appliance is **6 CPU** and **12 GB** of RAM.

=== "Online"

    1. Install microk8s:
    ```bash
    sudo snap install microk8s --classic
    ```
    2. Install microk8s addons:
    ```bash
    for addon in "ingress" "hostpath-storage" "metrics-server"; do
        sudo microk8s enable "$addon"
    done
    ```
    3. Install Helm and set it up to use with microk8s:
    ```bash
    sudo snap install helm --classic
    sudo mkdir /var/snap/microk8s/current/bin
    sudo ln -s /snap/bin/helm /var/snap/microk8s/current/bin/helm
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
        ```
        Fetch  container images. Be sure to check you have the correct image tags as specified [microk8s-community-addons](https://github.com/canonical/microk8s-community-addons/tree/main/addons)
        === "All-in-One"
            ```bash
            # Calico CNI
            for container_image in "cni" "pod2daemon-flexvol" "node"
            do
                docker pull calico/$container_image:v3.19.1 && docker save calico/$container_image:v3.19.1 >> calico_$container_image.tar
            done
            docker pull calico/kube-controllers:v3.17.3 && docker save calico/kube-controllers:v3.17.3 >> calico_kube-controllers.tar

            # microk8s add-ons, refer to images used in corresponding .yaml files (https://github.com/ubuntu/microk8s/tree/master/microk8s-resources/actions)
            export ARCH=amd64

            # DNS
            for container_image in "k8s-dns-kube-dns" "k8s-dns-dnsmasq-nanny" "k8s-dns-sidecar"
            do
                docker pull gcr.io/google_containers/$container_image-$ARCH:1.14.7 && docker save gcr.io/google_containers/$container_image-$ARCH:1.14.7 >> $container_image.tar
            done

            #coreDNS, storage, metrics-server, ingress, registry
            docker pull k8s.gcr.io/pause:3.1 && docker save k8s.gcr.io/pause:3.1 >> pause.tar
            docker pull coredns/coredns:1.8.0 && docker save coredns/coredns:1.8.0 >> coredns.tar
            docker pull cdkbot/hostpath-provisioner-$ARCH:1.0.0 && docker save cdkbot/hostpath-provisioner-$ARCH:1.0.0 >> storage.tar
            docker pull k8s.gcr.io/metrics-server/metrics-server:v0.5.2 && docker save k8s.gcr.io/metrics-server/metrics-server:v0.5.2 >> metrics.tar
            docker pull k8s.gcr.io/ingress-nginx/controller:v1.1.0 && docker save k8s.gcr.io/ingress-nginx/controller:v1.1.0 >> ingress.tar
            docker pull registry:2.7.1 && docker save registry:2.7.1 >> registry.tar


            # Assemblyline Core (release: 4.7.stable)
            for al_image in "core" "ui" "ui-frontend" "service-server" "socketio" "rust"
            do
                docker pull cccs/assemblyline-$al_image:4.7.stable && docker save cccs/assemblyline-$al_image:4.7.stable >> al_$al_image.tar
            done

            # Elastic
            ES_REG=docker.elastic.co
            ES_VER=7.17.3
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

            #Pull official service images
            for svc_image in apivector apkaye batchdeobfuscator capa cape characterize configextractor deobfuscripter elf elfparser emlparser espresso extract floss frankenstrings iparse metapeek oletools pdfid peepdf pe pixaxe safelist sigma suricata swiffer tagcheck torrentslicer unpacme unpacker vipermonkey virustotal-dynamic virustotal-static xlmmacrodeobfuscator yara
            do
                docker pull cccs/assemblyline-service-$svc_image:stable && docker save cccs/assemblyline-service-$svc_image:stable >> $svc_image.tar
            done
            ```
        === "Step-by-Step"
            === "Assemblyline"
                ```bash
                # Core
                for al_image in "core" "ui" "ui-frontend" "service-server" "socketio"
                do
                    docker pull cccs/assemblyline-$al_image:4.2.stable && docker save cccs/assemblyline-$al_image:4.2.stable >> al_$al_image.tar
                done

                # Services
                for svc_image in apivector apkaye batchdeobfuscator capa cape characterize configextractor deobfuscripter elf elfparser emlparser espresso extract floss frankenstrings iparse metapeek oletools pdfid peepdf pe pixaxe safelist sigma suricata swiffer tagcheck torrentslicer unpacme unpacker vipermonkey virustotal-dynamic virustotal-static xlmmacrodeobfuscator yara
                do
                    docker pull cccs/assemblyline-service-$svc_image:stable && docker save cccs/assemblyline-service-$svc_image:stable >> $svc_image.tar
                done
                ```
            === "AL Helm Chart Dependencies"
                === "Elastic"
                    ```bash
                    ES_REG=docker.elastic.co
                    ES_VER=7.16.2
                    for beat in "filebeat" "metricbeat"
                    do
                        docker pull $ES_REG/beats/$beat:$ES_VER && docker save $ES_REG/beats/$beat:$ES_VER >> es_$beat.tar
                    done
                    for es in "logstash" "kibana" "elasticsearch"
                    do
                        docker pull $ES_REG/$es/$es:$ES_VER && docker save $ES_REG/$es/$es:$ES_VER >> es_$es.tar
                    done
                    ```
                === "MinIO"
                    ```bash
                    for minio_image in "minio:RELEASE.2021-02-14T04-01-33Z" "mc:RELEASE.2021-02-14T04-28-06Z"
                    do
                        docker pull minio/$minio_image && docker save minio/$minio_image >> minio_$minio_image.tar
                    done
                    ```
                === "Redis"
                    ```bash
                    docker pull redis && docker save redis >> redis.tar
                    ```
            === "Calico CNI"
                ```bash
                for container_image in "cni" "pod2daemon-flexvol" "node"
                do
                    docker pull calico/$container_image:v3.19.1 && docker save calico/$container_image:v3.19.1 >> calico_$container_image.tar
                done
                docker pull calico/kube-controllers:v3.17.3 && docker save calico/kube-controllers:v3.17.3 >> calico_kube-controllers.tar
                ```
            === "MicroK8s addons"
                Refer to: [microk8s-community-addons](https://github.com/canonical/microk8s-community-addons/tree/main/addons)
                ```bash
                # Kubernetes DNS
                for container_image in "k8s-dns-kube-dns" "k8s-dns-dnsmasq-nanny" "k8s-dns-sidecar"
                do
                    docker pull gcr.io/google_containers/$container_image-$ARCH:1.14.7 && docker save gcr.io/google_containers/$container_image-$ARCH:1.14.7 >> $container_image.tar
                done

                #coreDNS, storage, metrics-server, ingress, registry
                docker pull k8s.gcr.io/pause:3.1 && docker save k8s.gcr.io/pause:3.1 >> pause.tar
                docker pull coredns/coredns:1.8.0 && docker save coredns/coredns:1.8.0 >> coredns.tar
                docker pull cdkbot/hostpath-provisioner-$ARCH:1.0.0 && docker save cdkbot/hostpath-provisioner-$ARCH:1.0.0 >> storage.tar
                docker pull k8s.gcr.io/metrics-server/metrics-server:v0.5.2 && docker save k8s.gcr.io/metrics-server/metrics-server:v0.5.2 >> metrics.tar
                docker pull k8s.gcr.io/ingress-nginx/controller:v1.1.0 && docker save k8s.gcr.io/ingress-nginx/controller:v1.1.0 >> ingress.tar
                docker pull registry:2.7.1 && docker save registry:2.7.1 >> registry.tar
                ```

    2. Setup master node
        1. Install microk8s
            ```bash
            sudo snap ack microk8s_*.assert
            sudo snap install microk8s_*.snap --classic
            ```

        2. Setup registry addon (Optional)
            ```bash
            # Add registry image to microk8s
            microk8s.ctr image pull registry.tar
            microk8s addon registry
            ```

        3. Load images from disk and push to registry
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

        4.  Modify Container Registry Endpoints
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
        5. Enable microk8s add-ons
            ```bash
            sudo microk8s reset && sudo microk8s enable ingress dns ha-cluster storage metrics-server
            ```
        6. Fetch kubeconfig for administration
            ```bash
            cp /var/snap/microk8s/current/credentials/client.config <remote_destination>
            ```

        7. Install admin tools (separate or same host(s) for computing):
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

            # (Optional) Install additional Kubernetes monitoring tools like k9s or FreeLens
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

## Configuring your deployment

We will assume you are creating a directory named "deployment" where all referenced files are stored.

```bash
mkdir deployment; cd deployment
wget https://raw.githubusercontent.com/CybercentreCanada/assemblyline-helm-chart/refs/heads/main/appliance/values.yaml
wget https://raw.githubusercontent.com/CybercentreCanada/assemblyline-helm-chart/refs/heads/main/appliance/secrets.yaml
```

The ```values.yaml``` file in your deployment directory is already pre-configured for use with microk8s as a basic one node minimal appliance. Make sure you go through the file to adjust disk sizes and to turn on/off features to your liking. If you wish to login to assemblyline from a bare public IP address, the configuration still requires a FQDN. As mentioned in `values.yaml`, `nip.io` can provide an easy FQDN for a bare IP.

```bash
sed -i "s/fqdn: \"localhost\"/fqdn: \"$(curl -s https:\/\/api.ipify.org).nip.io\"/" ~/git/deployment/values.yaml
```

The ```secret.yaml``` file in your deployment directory is preconfigured with default passwords, you should change them.

!!! tip
    The secrets are used to set up during bootstrap so make sure you change them before deploying the ```al``` chart.
!!! warning
    Be sure to update any values as deemed necessary ie. FQDN

## Deploy Assemblyline via Helm

### Add the assemblyline chart repository to helm

```bash
helm repo add assemblyline https://cybercentrecanada.github.io/assemblyline-helm-chart/
```

You may need to update the chart repositories to see the latest versions.

```bash
helm repo update
```

### Create a namespace for your Assemblyline install

For this documentation, we will use ```al``` as the namespace.

```bash
sudo microk8s kubectl create namespace al
```

### Deploy the secret to the namespace

```bash
sudo microk8s kubectl apply -f secrets.yaml --namespace=al
```

From this point on, you don't need the secrets.yaml file anymore. You should delete it so there is no file on disk containing your passwords.

```bash
rm secrets.yaml
```

### Finally, let's deploy Assemblyline's chart

For documentation, we will use ```assemblyline``` as the deployment name.

```bash
sudo microk8s helm install assemblyline assemblyline/assemblyline -f values.yaml -n al
```

!!! tip
    A helpful command for watching the containers as they become ready and running is `watch -n 1 microk8s kubectl get pods -n al`.

!!! warning
    After you've ran the `helm install` command, the system has a lot of setting up to do (Creating database indexes, loading service, setting up default accounts, loading signatures ...). Don't expect it to be fully operational for at least the next 15 minutes.

## Updating the current deployment

For details on updating your depyloment via helm see the [update instructions in the kubernetes cluster documentation.](../../cluster/general#update-your-deployment)

## Quality of life improvements

### [Cluster Management Tools](../../cluster/general#6-optional-cluster-management-tools)

### Sudoless access to MicroK8s

MicroK8s require you to add sudo in front of every command, you can add your user to the microk8s group so you don't have to.

```bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```

!!! warning "You will need to reboot for these changes to take effect"
    Temporarily, you can add the group to your current shell by running the following:
    ```bash
    newgrp microk8s
    ```

### Alias to Kubectl

Since all is running inside microk8s you can create an alias to the kubectl command to make your life easier

```bash
sudo snap alias microk8s.kubectl kubectl
kubectl config set-context --current --namespace=al
```

## Alternative Installations

We will officially only support microk8s installations for appliances, but you can technically install it on any local Kubernetes frameworks (k3s, minikube, kind...). That said there will be no documentation for these setups, and you will have to modify the ```values.yaml``` storage classes to fit with your desired framework.
