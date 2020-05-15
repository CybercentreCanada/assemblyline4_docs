---
layout: default
title: Local development
parent: Infrastructure
grand_parent: Developer's manual
has_children: false
has_toc: false
nav_order: 2
---

# Local development
{: .no_toc }

---

This documentation will show you how to setup you development Virtual Machine for local development which means that you will run you IDE inside the virtual machine. This is by far the easiest setup to get working and removes a lot of headaches.

## Operating system 

For this documentation, we will assume that you are working on a fresh installation of [Ubuntu 18.04 Desktop](http://releases.ubuntu.com/18.04.4/ubuntu-18.04.4-desktop-amd64.iso)

### Update VM

Make sure ubuntu is running the latest software

    sudo apt update
    sudo apt dist-upgrade
    sudo snap refresh

Make sure you run updated kernel

    sudo apt install -y linux-image-generic-hwe-18.04

## Installing pre-requisite softwares

### Install Assemblyline APT dependencies

    sudo apt update
    sudo apt install -y libffi6 libfuzzy2 libmagic1 build-essential libffi-dev libfuzzy-dev libldap-2.4-2 libsasl2-2 libldap2-dev libsasl2-dev

### Install Python 3.7

Assemblyline 4 works on only python 3.7 and up. Also, the containers that we build target python 3.7 therefor we will install Python 3.7.

    sudo apt install -y python3-venv python3.7 python3.7-dev python3.7-venv

### Installing Docker

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

### Installing docker-compose

Installing docker-compose is done the same way on all Linux distros. Follow these simple instructions:

    # Install docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    
    # Test docker-compose installation
    docker-compose --version

### Installing Pycharm

We are using Pycharm Pro to to our development but since you have chosen local dev, you will not need the remote deployment features. Therefor you can use the free Pycharm community edition.

    sudo snap install --classic pycharm-community

## Adding Assemblyline specific configuration 

### Folders

Because Assemblyline use it's own set of folders inside the core, service-server and UI container, we have to create the same folder structure here so we can run the components in debug mode.

    sudo mkdir -p /etc/assemblyline
    sudo mkdir -p /var/cache/assemblyline
    sudo mkdir -p /var/lib/assemblyline
    sudo mkdir -p /var/log/assemblyline

    sudo chown $USER /etc/assemblyline
    sudo chown $USER /var/cache/assemblyline
    sudo chown $USER /var/lib/assemblyline
    sudo chown $USER /var/log/assemblyline

### Dev default configuration files

Here we will create configuration files that matches the default dev docker-compose configuration files so we can swap any of the components to one that is being debugged.

    echo "enforce: true" > /etc/assemblyline/classification.yml
    echo "
    auth:
      internal:
        enabled: true

    core:
      alerter:
        delay: 0
      metrics:
        apm_server:
          server_url: http://localhost:8200/
        elasticsearch:
          hosts: [http://elastic:devpass@localhost]

    datastore:
      ilm:
        indexes:
          alert:
            unit: m
          error:
            unit: m
          file:
            unit: m
          result:
            unit: m
          submission:
            unit: m

    filestore:
      cache:
        - file:///var/cache/assemblyline/

    logging:
      log_level: INFO
      log_as_json: false

    ui:
      audit: false
      debug: false
      enforce_quota: false
      fqdn: 127.0.0.1.nip.io
    " > /etc/assemblyline/config.yml

**NOTE:** As you can see in the last command we are setting the FQDN to 127.0.0.1.nip.io. NIP.IO is a service that will resolv the first part of the domain **127.0.0.1**.nip.io to it's IP value. We use this to fake DNS when there are none. This is especially useful for oAuth because some providers are forbidding redirect urls to IPs.

## Setup Assmblyline source

### Install git

Since your VM running Unbuntu 18.04 you can just install it with APT:

    sudo apt install -y git

*(Optional)* You should add your desktop SSH keys to your github account to use git via SSH. Follow these instructions to do so: [Github Help](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

### Clone core repositories

    # Create the core working directory
    mkdir -p ~/git/alv4
    cd ~/git/alv4

    # Clone repositories using HTTPS if you don't have your github account configured with an SSH key
    git clone https://github.com/CybercentreCanada/assemblyline-base.git
    git clone https://github.com/CybercentreCanada/assemblyline-core.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-client.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-server.git
    git clone https://github.com/CybercentreCanada/assemblyline-ui.git
    git clone https://github.com/CybercentreCanada/assemblyline-v4-service.git
    
    # OR Via SSH if you have your ssh id_rsa file configured to your github account
    git clone git@github.com:CybercentreCanada/assemblyline-base.git 
    git clone git@github.com:CybercentreCanada/assemblyline-core.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-client.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-server.git
    git clone git@github.com:CybercentreCanada/assemblyline-ui.git
    git clone git@github.com:CybercentreCanada/assemblyline-v4-service.git

### Setting up Core Virtualenv

    # Directly in the alv4 source directory 
    cd ~/git/alv4

    # Create the virtualenv and activate it
    python3.7 -m venv venv
    source ~/git/alv4/venv/bin/activate

    # Install assemblyline packages with their test dependencies
    pip install assemblyline[test] assemblyline-core[test] assemblyline-service-server[test] assemblyline-ui[test]

    # Remove assemblyline packages because we will use the live code
    pip uninstall -y assemblyline assemblyline-core assemblyline-service-server assemblyline-ui

### Setup Pycharm 

#### Load Alv4 Project

 1. Load **pycharm-community**
    - Choose whatever configuration option you want until the **Welcome screen**
 2. Click the **Open** button
 3. Choose the **~/git/alv4 directory**

 Your python interpreter should already be loaded because pycharm will take ./venv directory as the project interpreter by default. Let it load the interpreter...

#### Setup project structure

 1. Click **Files** -> **Settings**
 2. Select **Project: alv4** -> **Python Structure**
 3. For each top level folder (**assemblyline-base**, **assemblyline-core**...)
    - Click on it
    - Click **Source**
 4. Click **apply** then **OK**

#### Setup link to docker

Pycharm Community edition does not have a Docker integration. Instead you will have to run the docker and docker-compose commands by hand.
