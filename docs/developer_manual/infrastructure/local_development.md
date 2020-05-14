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

    sudo mkdir -p /etc/assemblyline
    sudo mkdir -p /var/cache/assemblyline
    sudo mkdir -p /var/lib/assemblyline
    sudo mkdir -p /var/log/assemblyline

    sudo chown $USER /etc/assemblyline
    sudo chown $USER /var/cache/assemblyline
    sudo chown $USER /var/lib/assemblyline
    sudo chown $USER /var/log/assemblyline

### Dev default configuration files

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