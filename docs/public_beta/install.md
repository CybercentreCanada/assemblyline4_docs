---
layout: default
title: Install Beta
parent: Public Beta
nav_order: 1
has_children: false
has_toc: false
---

# Installing Public Beta
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Appliance Installation

For this first public beta, the only installation we will support is the appliance installation (This can also be used for service development).

In this setup, AssemblyLine 4 will use all of the resources available on your machine for it's components. This can be deployed on any machine / VM that can run Docker containers. 

### Pre-requisites

We recommend that you run AssemblyLine on a system with the following minimal specs:

    Any linux distribution with a recent kernel (4.4+)
    4 Cores
    8 GB of Ram
    40 GB of disk space

#### Docker pre-installed on your machine
Appliance mode will use Docker to start Assemblyline therefore it will need to be installed on your machine. 

NOTE: `All instructions that follow need to be performed on the machine where Assemblyline will be installed.`

##### Installation of docker on Ubuntu (Recommended)
Follow these simple commands to get docker runnning on your machine:

    # Add docker repository
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo apt-key fingerprint 0EBFCD88
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

    # Install docker
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io

    # Test docker installation
    sudo docker run hello-world

##### Installation of Docker on other linux distro

Follow instructions on Docker's website: [https://docs.docker.com/install/](https://docs.docker.com/install/)

#### Add your proxy settings to docker (Optional)

Create `$HOME/.docker/config.json` file:

    mkdir -p ~/.docker/
    nano ~/.docker/config.json

    # Save your proxy settings inside the file
    {
        "proxies":
        {
            "default":
            {
                "httpProxy": "http://<PROXY_ADDRESS>:<PROXY_PORT>/",
                "httpsProxy": "http://<PROXY_ADDRESS>:<PROXY_PORT>/",
                "noProxy": "<YOUR_CUSTOM_PROXY_EXEMPTIONS>,localhost,127.0.0.1,minio,elasticsearch,redis,nginx,al_service_server,service-server,al_ui,al_socketio"
            }
        }
    }

Create systemd proxy setting file for docker:

    nano /etc/systemd/system/docker.service.d/https-proxy.conf

    # Save your proxy settings in the file

    [Service]

    Environment="HTTPS_PROXY=http://<PROXY_ADDRESS>:<PROXY_PORT>/" "HTTP_PROXY=http://<PROXY_ADDRESS>:<PROXY_PORT>/" "NO_PROXY=<YOUR_CUSTOM_PROXY_EXEMPTIONS>,localhost,127.0.0.1,minio,elasticsearch,redis,nginx,al_service_server,service-server,al_ui,al_socketio"

Reload docker and test:

    # Reload docker 
    sudo systemctl daemon-reload
    sudo systemctl restart docker

    # Test systemd
    systemctl show --property=Environment docker

    # Test docker containers
    docker container run --rm busybox env

#### Docker-compose pre-installed on your computer
Installing docker-compose is done the same way on all Linux distros. Follow these simple instructions:

    # Install docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    
    # Test docker-compose installation
    docker-compose --version

For reference, here are the instructions on Docker's website: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

### Download installation files

Download installation files to your home directory:

    curl -L "https://assemblyline-support.s3.amazonaws.com/assemblyline4_beta_1.tar.gz" -o $HOME/assemblyline4_beta_1.tar.gz

Or you can manually download them and put them in the right spot:

[Download Beta](https://assemblyline-support.s3.amazonaws.com/assemblyline4_beta_1.tar.gz){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

### Extract installation files

From now on, we will assume you've downloaded the file in your home directory.

Decompress the installation archive:

    tar zxvf $HOME/assemblyline4_beta_1.tar.gz

### Configure system

Generate a self-signed cert for your installation:

    openssl req -nodes -x509 -newkey rsa:4096 -keyout $HOME/assemblyline4_beta_1/config/nginx.key -out $HOME/assemblyline4_beta_1/config/nginx.crt -days 365 -subj "/C=CA/ST=Ontario/L=Ottawa/O=CCCS/CN=assemblyline.local"


(`Optional`) Edit the default passwords in the following two files:
    
    nano $HOME/assemblyline4_beta_1/.env
    nano $HOME/assemblyline4_beta_1/config/bootstrap.py


### Run docker_compose file
Every time you run docker-compose commands you must be in the directory were the compose file resides.

    cd $HOME/assemblyline4_beta_1/

Now you can start Assemblyline Beta

    sudo docker-compose up -d

### Assemblyline is started, now what? 

There is a few things you need to know now that you have an Assemblyline4 instance started

1. The bootstrap process for the first boot may take about 5 minutes to make sure everything is ready. 
2. The yara service will start in an update state and will pull all yara rules from [https://github.com/Yara-Rules/rules](https://github.com/Yara-Rules/rules). There are about 12000+ rules to process which will take some time. Also, at the same time, Suricata will pull all 25000+ `emerging threats` signatures. The process of pulling the signatures for those two services can take up to 30 minutes. This will be improved in the future to make it faster. This process is repeated automatically every 24 hours to keep the service rules in sync.
3. To get access to your beta instance just browse to `https://<IP_OF_YOUR_AL_INSTANCE>`. The default username/password to access the instance is `admin`/`admin`. (Note: If you changed the password in the `bootstrap.py` file, use this password instead)

#### Viewing logs
There is no logs centralization with this deployment but because everything runs on the same box you can leverage docker logging infrastructure.

To view logs for the core components:

    # From your docker-compose directory
    cd $HOME/assemblyline4_beta_1/

    # View all core-component logs
    sudo docker-compose logs

    # View logs for specific core component
    sudo docker-compose logs al_ui

    # Tail logs for a core component
    sudo docker-compose logs -f al_ui

Services are not loaded from your docker compose file, they are loaded by a component called `scaler`. To view their logs you must use docker directly.

    # list all loaded services
    sudo docker ps | grep "\-service\-"

    # View logs for a specific service
    sudo docker logs al_YARA_0

    # Tail logs for a sevice 
    sudo docker logs -f al_YARA_0

    # Going through a big log file using less
    sudo docker logs al_YARA_0 2>&1 | less

#### Access the Assemblyline CLI

To load the `Assemblyline CLI` you will need to hijack on of the running container. We recommended using the `updater` container.

    sudo docker exec -it assemblyline4_beta_1_al_updater_1 python -m assemblyline.run.cli
