# Appliance installation (Docker)

This is the documentation for an appliance instance of the Assemblyline platform suited for very small single machine deployment.

## Setup requirements

!!! info "Caveat"
    The documentation provided here assumes that you are installing your appliance on an Ubuntu-based system and was only tested on Ubuntu 20.04. You might have to change the commands a bit if you use other Linux distributions.

    The recommended minimum system requirement for this appliance is **4 CPUs** and **8 GB** of Ram.

!!! warning
    If you have more then **16 CPUs** and **64 GB** of ram, you should consider using the [Microk8s appliance](../appliance/) instead. Microk8s will be able to auto-scale core components based on load but this docker appliance can only scale services.

### Install pre-requisites
=== "Online"
    === "Ubuntu"
        1. Install Docker:
        ```bash
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
        ```
        ```bash
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        ```
        ```bash
        sudo apt-key fingerprint 0EBFCD88
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        sudo apt-get install -y docker-ce docker-ce-cli containerd.io
        ```
        2. Install Docker compose:
        ```bash
        sudo curl -s -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
        sudo chmod +x /usr/local/bin/docker-compose
        sudo curl -s -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
        ```
    === "RHEL 8.5"
        !!! warning
            Many of the instructions below have been set to force yes and allowerasing for quick implementation.

            It is recommended that these flags be removed for production environments to avoid impacting production environments by missing key messages and warnings.
            Step 4 contains a firewall configuration, we strongly advise firewall settings should be managed and reviewed by your organization before deployment.

        1. Install Docker:
        ```bash
        yum update -y --allowerasing
        yum install -y yum-utils
        yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        yum install -y docker-ce docker-ce-cli containerd.io --allowerasing
        systemctl start docker
        systemctl enable docker
        ```

        2. Install Docker compose:
        ```bash
        curl -s -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
        chmod +x /usr/bin/docker-compose
        curl -s -L https://raw.githubusercontent.com/docker/compose/1.29.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
        ```

        3. Upgrade Python3.9:
        ```bash
        dnf install -y python39
        alternatives --set python3 /usr/bin/python3.9
        python3 --version
        ```

        4. Configure firewalld for Docker:
        ```bash
        sed -i 's/FirewallBackend=nftables/FirewallBackend=iptables/' /etc/firewalld/firewalld.conf
        firewall-cmd --reload
        reboot
        ```


=== "Offline"

    TBD

## Setup your Assemblyline appliance

### Download the Assemblyline docker-compose files

=== "Online"

    ```bash
    mkdir ~/git
    cd ~/git
    git clone https://github.com/CybercentreCanada/assemblyline-docker-compose.git
    ```

=== "Offline"

    TBD

### Choose your deployment type

!!! important
    After this step, we will assume that the commands that you run are from your deployment directory: ```~/deployments/assemblyline/```

=== "Assemblyline only"

    ```bash
    mkdir ~/deployments
    cp -R ~/git/assemblyline-docker-compose/minimal_appliance ~/deployments/assemblyline
    cd ~/deployments/assemblyline
    ```

=== "Assemblyline with ELK monitoring stack"

    ```bash
    mkdir ~/deployments
    cp -R ~/git/assemblyline-docker-compose/full_appliance ~/deployments/assemblyline
    cd ~/deployments/assemblyline
    ```

### Setup your appliance

The ```config/config.yaml``` file in your deployment directory is already pre-configured for use with docker-compose as a single node appliance. You can review the settings already configured but you should not have anything to change there.

The ```.env``` file in your deployment directory is preconfigured with default passwords, you should definitely change them.

## Deploy Assemblyline

### Create your https certs

```bash
openssl req -nodes -x509 -newkey rsa:4096 -keyout ~/deployments/assemblyline/config/nginx.key -out ~/deployments/assemblyline/config/nginx.crt -days 365 -subj "/C=CA/ST=Ontario/L=Ottawa/O=CCCS/CN=assemblyline.local"
```

### Pull necessary docker containers

```bash
cd ~/deployments/assemblyline
sudo docker-compose pull
sudo docker-compose build
sudo docker-compose -f bootstrap-compose.yaml pull
```

### Finally deploy your appliance

```bash
cd ~/deployments/assemblyline
sudo docker-compose up -d
sudo docker-compose -f bootstrap-compose.yaml up
```

!!! info
    Once the docker-compose command on the bootstrap file complete, your cluster will be ready to use and you can login with the default admin user/password that you've set in your ```.env``` file


## Docker Compose cheat sheet

### Updating your appliance

```bash
cd ~/deployments/assemblyline
sudo docker-compose pull
sudo docker-compose build
sudo docker-compose up -d
```

### Changing Assemblyline configuration file

Edit the ```cd ~/deployments/assemblyline/config/config.yml``` then:

```bash
cd ~/deployments/assemblyline
sudo docker-compose restart
```

### Check core services logs

For core components:
```bash
cd ~/deployments/assemblyline
sudo docker-compose logs
```
Or for a specific component:
```bash
cd ~/deployments/assemblyline
sudo docker-compose logs ui
```

### Take down your appliance

!!! tip
    This will remove all containers related to your appliance but will not remove the volumes so you can bring it back up safely.

```bash
cd ~/deployments/assemblyline
sudo docker-compose stop
sudo docker rm --force $(sudo docker ps -a --filter label=app=assemblyline -q)
sudo docker-compose down --remove-orphans
```

### Bring your appliance back online

```bash
cd ~/deployments/assemblyline
sudo docker-compose up -d
```
