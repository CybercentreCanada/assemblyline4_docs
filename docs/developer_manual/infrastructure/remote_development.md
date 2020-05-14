---
layout: default
title: Remote development
parent: Infrastructure
has_children: false
has_toc: false
nav_order: 3
---

# Remote development
{: .no_toc }

---
This documentation will show you how to setup your target Virtual Machine for remote development which means that you will run you IDE on your desktop and run the Assemblyline containers on the remote target VM.

## Operating system 

For this documentation, we will assume that you are working on a fresh installation of [Ubuntu 18.04 Server](http://releases.ubuntu.com/18.04.4/ubuntu-18.04.4-live-server-amd64.iso)

### Update traget VM

Make sure ubuntu is running the latest software

    sudo apt update
    sudo apt dist-upgrade

Make sure you run updated kernel

    sudo apt install -y linux-image-generic-hwe-18.04

Reboot 

    sudo reboot

## Installing pre-requisite softwares

### Install SSH Daemon

We need to make sure the remote target has SSH daemon installed for remote debugging

    sudo apt install -y ssh

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
