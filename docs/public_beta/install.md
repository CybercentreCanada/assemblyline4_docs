---
layout: default
title: Install Beta
parent: Public Beta
nav_order: 1
has_children: false
has_toc: false
---

# Installing Public Beta 1
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Appliance

For this first public beta, the only installation we will support is the appliance installation. (This can also be used for service development)

In this setup, Assemblyline 4 will use all the resources available on your machine for it's components. This can be deploy on any machine / VM that can run docker containers. 

### Pre-requisites

We recommend that you run Assemblyline on a system with the following specs:

    Any linux distribution with a recent kernel (4.4+)
    4 Cores
    8 GB of Ram
    40 GB of disk space

Install docker on your machine: [https://docs.docker.com/install/](https://docs.docker.com/install/)

Install docker_compose on your machine: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

### Download installation files

Installation files refered in this documentation are available to download from our Amazon AWS repo.

    wget https://assemblyline-support.s3.amazonaws.com/assemblyline4-beta-1.tar.gz

[Download](https://assemblyline-support.s3.amazonaws.com/assemblyline4-beta-1.tar.gz){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

### Extract installation files

Let's assume you've downloaded the file in your home directory.

    tar zxvf $HOME/assemblyline4-beta-1.tar.gz

### Configure system

Generate a self signed cert for your installation

    openssl req -nodes -x509 -newkey rsa:4096 -keyout $HOME/assemblyline4-beta-1/config/nginx.key -out $HOME/assemblyline4-beta-1/config/nginx.crt -days 365

Edit the default passwords in the following two files:
    
    $HOME/assemblyline4-beta-1/.env
    $HOME/assemblyline4-beta-1/config/bootstrap.py


### Run docker_compose file

    (cd $HOME/assemblyline4-beta-1/ && sudo docker-compose up -d)