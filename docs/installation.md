---
layout: default
title: Installation
nav_order: 2
has_children: true
has_toc: true
---

<img src="./images/al4.png" width="725">

There are many aspects to deploying an Assemblyline system that should be considered.
Although there are usually reasonable defaults, one should still be aware.

- Will you need a cluster of computers or a single node for processing files?
    - There are two deployment methods, a cluster running kubernetes, or a single node appliance running a daemon docker.

- Do you have an ELK stack collecting logs already that Assemblyline should feed into?
    - By default cluster deployments include their own logging ELK stack, and appliances do not, but both deployment methods can have their logging customized.

- Where should the files being processed by Assemblyline be stored? 
    - Running in the cloud, Assemblyline can be configured to use blob storage for your cloud provider.
    - On premises FTP servers or private S3 compatible servers can be used.
    - Private deployments of Assemblyline can simply use a local disk.

- Are you going to integrate with an external authentication system?
    - The default authentication uses an internal user database, which can be replaced or augmented by LDAP or OAuth servers.
  
 
