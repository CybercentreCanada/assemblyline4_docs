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

## Appliance

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

##### Installation of docker on Ubuntu (Recommended)
Follow these simple commands to get docker runnning on your machine:

    # Add docker repository
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo apt-key fingerprint 0EBFCD88
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

    # Install docker
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io

    # Test docker installation
    sudo docker run hello-world

##### Installation of Docker on other linux distro

Follow instructions on Docker's website: [https://docs.docker.com/install/](https://docs.docker.com/install/)

#### Docker-compose pre-installed on your computer
Installing docker-compose is done the same way on all Linux distros. Follow these simple instructions:

    # Install docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    
    # Test docker-compose installation
    docker-compose --version

For reference, here are the instructions on Docker's website: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

### Download installation files

Installation files referred to in this documentation are available for download from our Amazon AWS repo. Let's download them to your home directory.

    wget https://assemblyline-support.s3.amazonaws.com/assemblyline4_beta_1.tar.gz -o $HOME/assemblyline4_beta_1.tar.gz

[Download Beta](https://assemblyline-support.s3.amazonaws.com/assemblyline4_beta_1.tar.gz){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

### Extract installation files

Let's assume you've downloaded the file in your home directory.

    tar zxvf $HOME/assemblyline4_beta_1.tar.gz

### Configure system

Generate a self-signed cert for your installation:

    openssl req -nodes -x509 -newkey rsa:4096 -keyout $HOME/assemblyline4_beta_1/config/nginx.key -out $HOME/assemblyline4_beta_1/config/nginx.crt -days 365 -subj "/C=CA/ST=Ontario/L=Ottawa/O=CCCS/CN=assemblyline.local"


Edit the default passwords in the following two files:
    
    nano $HOME/assemblyline4_beta_1/.env
    nano $HOME/assemblyline4_beta_1/config/bootstrap.py


### Run docker_compose file
Every time you run docker-compose commands you must be in the directory were the compose file resides

    cd $HOME/assemblyline4_beta_1/

Now you can start Assemblyline Beta

    sudo docker-compose up -d