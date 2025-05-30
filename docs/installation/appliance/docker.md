# Appliance in Docker

This is the documentation for an appliance instance of the Assemblyline platform suited for very small single machine deployment.

## Setup requirements

!!! info "Caveat"
    The documentation provided here assumes that you are installing your appliance on one of the following systems:

    - Debian: Ubuntu 22.04, Ubuntu 24.04
    - RHEL: RHEL 8.5

    You might have to change the commands a bit if you use other Linux distributions.

    The recommended minimum system requirement for this appliance is **4 CPUs** and **8 GB** of RAM.

    **Note:** If you have above the minimum system requirement, you can make performance adjustments such as setting `core.scaler.service_defaults.min_instances: 1` to ensure service readiness by default rather than on-demand scaling.

### Install pre-requisites
=== "Ubuntu"
    Install Docker:
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(lsb_release -cs)" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose
    ```
=== "RHEL 8.5"
    !!! warning
        Many of the instructions below have been set to force yes and allow erasing for quick implementation.

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
```json
{
  "default-address-pools":
  [
    {"base":"10.201.0.0/16","size":24}
  ]
}
```

```bash
echo '{"default-address-pools":[{"base":"10.201.0.0/16","size":24}]}' | jq '.' | sudo tee /etc/docker/daemon.json
```

2. Restart Docker to acknowledge configuration: `service docker restart`

## Setup your Assemblyline appliance

### Download the Assemblyline Docker Compose repository

```bash
git clone https://github.com/CybercentreCanada/assemblyline-docker-compose.git ~/deployments/assemblyline
cd ~/deployments/assemblyline
```

### Setup your appliance

The ```config/config.yaml``` file in your deployment directory is already pre-configured for use with `docker-compose` as a single node appliance. You can review the settings already configured but you should not have anything to change there.

The ```.env``` file in your deployment directory is preconfigured with default passwords, you should definitely change them.

#### Assign profiles depending on your deployment requirements
These following [service profiles](https://docs.docker.com/compose/how-tos/profiles/) can be combined (unless otherwise specified) depending on your deployment requirements:

- `minimal`: This setup includes the bare-minimum components for everything to be able to run. There will be no metrics collected and you will have to tail the log from the docker container logs.
- `full`: This setup includes every single components and all metrics and logging capabilities. Metrics and logs will be gathered inside the same Elasticsearch instance as the processing data and you will have to access Kibana to view all of those.
- `archive`: This deploys the Archiver component of Assemblyline but this requires `datastore.archive.enabled: true` in your `config.yml` otherwise the container will terminate.

**Note**: The `minimal` and `full` profiles are mutually exclusive and are not to be used together.

You can specify which profiles to use on the commandline using the `--profile` flag or set `COMPOSE_PROFILES` in your `.env` file as a comma separated list (default: `COMPOSE_PROFILES=minimal`)

## Deploy Assemblyline

!!! info
    The following instructions assume `.env` contains `COMPOSE_PROFILES`, otherwise the `--profile` flag can be used to override what's set in `.env`

### Create your https certs

```bash
openssl req -nodes -x509 -newkey rsa:4096 -keyout ~/deployments/assemblyline/config/nginx.key -out ~/deployments/assemblyline/config/nginx.crt -days 365 -subj "/C=CA/ST=Ontario/L=Ottawa/O=CCCS/CN=assemblyline.local"
```

### Pull necessary Docker containers

```bash
cd ~/deployments/assemblyline
sudo docker-compose pull --ignore-buildable
sudo env COMPOSE_BAKE=true docker-compose build
sudo docker-compose -f bootstrap-compose.yaml pull
```

### Finally deploy your appliance

```bash
cd ~/deployments/assemblyline
sudo docker-compose up -d --wait
sudo docker-compose -f bootstrap-compose.yaml up
```

!!! tip
    A helpful command for watching the containers as they start and enter a healthy state is `watch -n 1 docker ps`.

!!! info
    Once the `docker-compose` command on the bootstrap file complete, your cluster will be ready to use and you can login with the default admin user/password that you've set in your ```.env``` file

### Service specific configurations

Certain services need special configurations to run efficiently in a docker-compose appliance setup. Refer to the [service management documentation](https://cybercentrecanada.github.io/assemblyline4_docs/administration/service_management/) to configure services through service parameters and service variables.

- URLDownloader: The `no_sandbox` option defaults to `False`, but will cause a `TimeoutExpired` in the internal google-chrome. Change this value to `True` if you encounter this [issue](https://github.com/CybercentreCanada/assemblyline/issues/146).


## Docker Compose cheat sheet

### Updating your appliance

```bash
cd ~/deployments/assemblyline
sudo docker-compose pull --ignore-buildable
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
