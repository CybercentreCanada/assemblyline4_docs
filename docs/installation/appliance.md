# Installing an Assemblyline appliance

## Appliance Installation

In this setup, Assemblyline 4 will use all of the resources available on your machine for its components. This can be deployed on any machine / VM that can run Docker containers. 
This can also be used for [service development](../developer_manual/assemblyline/getting_started/).

### Pre-requisites

We recommend that you run Assemblyline on a system with the following specs:

    Any linux distribution with a recent kernel (4.4+)
    32+ Cores
    64+ GB of Ram
    1TB+ of disk space

#### Docker pre-installed on your machine
Appliance mode will use Docker to start Assemblyline therefore it will need to be installed on your machine. 

NOTE: `All instructions that follow need to be performed on the machine where Assemblyline will be installed.`

##### Installation of Docker on Ubuntu (Recommended)
Follow these simple commands to get Docker running on your machine:

    # Add Docker repository
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo apt-key fingerprint 0EBFCD88
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

    # Install Docker
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io

    # Test Docker installation
    sudo docker run hello-world

##### Installation of Docker on other linux distro

Follow instructions on Docker's website: [https://docs.docker.com/install/](https://docs.docker.com/install/)

#### Add your proxy settings to Docker (Optional)

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

Create systemd proxy setting file for Docker:

    nano /etc/systemd/system/docker.service.d/https-proxy.conf

    # Save your proxy settings in the file

    [Service]

    Environment="HTTPS_PROXY=http://<PROXY_ADDRESS>:<PROXY_PORT>/" "HTTP_PROXY=http://<PROXY_ADDRESS>:<PROXY_PORT>/" "NO_PROXY=<YOUR_CUSTOM_PROXY_EXEMPTIONS>,localhost,127.0.0.1,minio,elasticsearch,redis,nginx,al_service_server,service-server,al_ui,al_socketio"

Reload Docker and test:

    # Reload Docker 
    sudo systemctl daemon-reload
    sudo systemctl restart docker

    # Test systemd
    systemctl show --property=Environment docker

    # Test Docker containers
    docker container run --rm busybox env

#### Docker-compose pre-installed on your computer
Installing docker-compose is done the same way on all Linux distros. Follow these simple instructions:

    # Install docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    
    # Test docker-compose installation
    docker-compose --version

For reference, here are the instructions on Docker's website: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

### Install an Assemblyline appliance

Follow the steps here: 

[https://github.com/CybercentreCanada/assemblyline-docker-compose#2-clone-this-repository](https://github.com/CybercentreCanada/assemblyline-docker-compose#2-clone-this-repository)

### Assemblyline is started, now what? 

There is a few things you need to know now that you have an Assemblyline4 instance started

1. The bootstrap process for the first boot may take about 5 minutes to make sure everything is ready. 
2. The services will slowly be added to the system while the `bootstrap-compose.yaml` file gets executed.
3. You do not have to wait 30 minutes anymore to get the list of signatures, this process now only take a minute.
4. To get access to your beta instance just browse to `https://<IP_OF_YOUR_AL_INSTANCE>`. The default username/password to access the instance is `admin`/`admin`. (Note: If you changed the password in the `bootstrap.py` file, use this password instead)

#### Viewing logs
There is no logs centralization with this deployment but because everything runs on the same box you can leverage Docker logging infrastructure.

To view logs for the core components:

    # From your docker-compose directory
    cd $HOME/assemblyline4_beta_4/

    # View all core-component logs
    sudo docker-compose logs

    # View logs for specific core component
    sudo docker-compose logs al_ui

    # Tail logs for a core component
    sudo docker-compose logs -f al_ui

Services are not loaded from your docker-compose file, they are loaded by a component called `scaler`. To view their logs you must use Docker directly.

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

    sudo docker exec -it assemblyline4_beta_4_al_updater_1 python -m assemblyline.run.cli

