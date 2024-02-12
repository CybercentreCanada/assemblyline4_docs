# Appliance in Docker

This is the documentation for an appliance instance of the Assemblyline platform suited for very small single machine deployment.

## Setup requirements

!!! info "Caveat"
    The documentation provided here assumes that you are installing your appliance on one of the following systems:

    - Debian: Ubuntu 20.04, Ubuntu 22.04
    - RHEL: RHEL 8.5

    You might have to change the commands a bit if you use other Linux distributions.

    The recommended minimum system requirement for this appliance is **4 CPUs** and **8 GB** of RAM.

### Install pre-requisites
=== "Online"
    === "Ubuntu"
        Install Docker:
        ```bash
        sudo apt-get update -y
        sudo apt-get install -y apt-transport-https ca-certificates curl gnupg software-properties-common
        sudo install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        sudo chmod a+r /etc/apt/keyrings/docker.gpg
        echo \
        "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update -y
        sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose
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
        yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
        yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin --allowerasing
        ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose
        systemctl start docker
        systemctl enable docker
        ```

        2. Upgrade Python3.9:
        ```bash
        dnf install -y python39
        alternatives --set python3 /usr/bin/python3.9
        python3 --version
        ```

        3. Configure firewalld for Docker:
        ```bash
        sed -i 's/FirewallBackend=nftables/FirewallBackend=iptables/' /etc/firewalld/firewalld.conf
        firewall-cmd --reload
        reboot
        ```

### Configure Docker to use larger address pools
1. Create/Edit `/etc/docker/daemon.json` and add the following lines:
```
{
  "default-address-pools":
  [
    {"base":"10.201.0.0/16","size":24}
  ]
}
```

2. Restart Docker to acknowledge configuration: `service docker restart`

## Setup your Assemblyline appliance

### Download the Assemblyline docker-compose files

=== "Online"

    ```bash
    mkdir ~/git
    cd ~/git
    git clone https://github.com/CybercentreCanada/assemblyline-docker-compose.git
    ```

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

    !!! warning
        Since everything is self-contained, you shouldn't need to install the ELK monitoring stack on the appliance.

    ```bash
    mkdir ~/deployments
    cp -R ~/git/assemblyline-docker-compose/full_appliance ~/deployments/assemblyline
    cd ~/deployments/assemblyline
    ```

### Setup your appliance

The ```config/config.yaml``` file in your deployment directory is already pre-configured for use with `docker-compose` as a single node appliance. You can review the settings already configured but you should not have anything to change there.

The ```.env``` file in your deployment directory is preconfigured with default passwords, you should definitely change them.

## Deploy Assemblyline

### Create your https certs

```bash
openssl req -nodes -x509 -newkey rsa:4096 -keyout ~/deployments/assemblyline/config/nginx.key -out ~/deployments/assemblyline/config/nginx.crt -days 365 -subj "/C=CA/ST=Ontario/L=Ottawa/O=CCCS/CN=assemblyline.local"
```

### Pull necessary Docker containers

```bash
cd ~/deployments/assemblyline
sudo docker-compose pull
# If you see the following error, have no fear, just run the next command (sudo docker-compose build)
# WARNING: Some service image(s) must be built from source by running:
#     docker compose build scaler updater
# 2 errors occurred:
#         * Error response from daemon: pull access denied for al_scaler, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
#         * Error response from daemon: pull access denied for al_updater, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
sudo docker-compose build
sudo docker-compose -f bootstrap-compose.yaml pull
```

### Finally deploy your appliance

```bash
cd ~/deployments/assemblyline
sudo docker-compose up -d --wait
sudo docker-compose -f bootstrap-compose.yaml up
```

!!! info
    Once the `docker-compose` command on the bootstrap file complete, your cluster will be ready to use and you can login with the default admin user/password that you've set in your ```.env``` file

### Service specific configurations

Certain services need special configurations to run efficiently in a docker-compose appliance setup. Refer to the [service management documentation](https://cybercentrecanada.github.io/assemblyline4_docs/administration/service_management/) to configure services through service parameters and service variables.

- URLDownloader: The `no_sandbox` option defaults to `False`, but will cause a `TimeoutExpired` in the internal google-chrome. Change this value to `True` if you encounter this [issue](https://github.com/CybercentreCanada/assemblyline/issues/146).


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
