---
layout: default
title: Local development
parent: Infrastructure
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
