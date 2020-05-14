---
layout: default
title: Getting Started
parent: Infrastructure
grand_parent: Developer's manual
has_children: false
has_toc: false
nav_order: 1
---

# Getting Started
{: .no_toc }

---

In this section, we will teach you how to setup your environment for developing on the core components of Assemblyline so you can debug them and run the code live without needing to create containers.

## Development Virtual Machine

Assemblyline core service require specific external packages and install files in specific directories in the containers. For this reason, it is recommended that you do no develop directly from your desktop unless you use remote debugging features. 

### Virtual Machine Minimum Specs

There is quite a few containers to run to spinup the whole Assemblyline infrastructure for development. For this reason, the development VM where the code will be running should have at least the following specs:

 - 2 Cores
 - 8 GB of RAM
 - 40 GB of disk space

## Operating system 

We recommend that you use Ubuntu 18.04 for your target machine, machine where the containers will be running. 

If you are going to use local development, running your IDE on the same box as the containers, using [Ubuntu Desktop](http://releases.ubuntu.com/18.04.4/ubuntu-18.04.4-desktop-amd64.iso).

Otherwise, if you are using remote debugging, your target machine should run the [Ubuntu server](http://releases.ubuntu.com/18.04.4/ubuntu-18.04.4-live-server-amd64.iso) because it will only be used to run the containers.
