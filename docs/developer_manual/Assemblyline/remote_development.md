---
layout: default
title: Remote development
parent: AssemblyLine
grand_parent: Developer's manual
has_children: false
has_toc: false
nav_order: 3
---

# Remote development
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

**NOTE:** To use this setup, we assume that you have a paid version of **Pycharm Professional** because we will be using features that are exclusive to the Pro version. If you don't, use the [Local development](local_development.html) setup instead.

This documentation will show you how to setup your target Virtual Machine for remote development which means that you will run you IDE on your desktop and run the Assemblyline containers on the remote target VM.

## Operating system 

For this documentation, we will assume that you are working on a fresh installation of [Ubuntu 18.04 Server](http://releases.ubuntu.com/18.04.4/ubuntu-18.04.4-live-server-amd64.iso)

### Update target VM

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

    sudo apt update
    sudo apt install -y ssh

### Install Assemblyline APT dependencies

    sudo apt update
    sudo apt install -y libffi6 libfuzzy2 libmagic1 build-essential libffi-dev libfuzzy-dev libldap-2.4-2 libsasl2-2 libldap2-dev libsasl2-dev

### Install Python 3.7

Assemblyline 4 works on only python 3.7 and up. Also, the containers that we build target python 3.7 therefor we will install Python 3.7.

    sudo apt install -y python3-venv python3.7 python3.7-dev python3.7-venv

### Installing docker-compose

Installing docker-compose is done the same way on all Linux distros. Follow these simple instructions:

    # Install docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    
    # Test docker-compose installation
    docker-compose --version

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

#### Securing docker for remote access

We are going to make your docker server accessible from the internet. To make it secure, we need to enable tls authentication in the docker daemon. Anywhere that you see assemblyline.local, you can change that value to your own DNS name. If your planning on using IP, you'll have to set a **static IP** to the remote VM because your cert will only allow conections to that IP.

    # Create a cert directory
    mkdir ~/certs
    cd ~/certs

    # Create a CA (Remember the password you've set)
    openssl genrsa -aes256 -out ca-key.pem 4096

    # Create a certificate signing request (ignore the .rng error)
    openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem -subj "/C=CA/ST=Ontario/L=Ottawa/O=CCCS/CN=assemblyline.local"

    # Creating the server public/private key
    openssl genrsa -out server-key.pem 4096
    openssl req -subj "/CN=assemblyline.local" -sha256 -new -key server-key.pem -out server.csr
    echo subjectAltName = DNS:assemblyline.local,IP:`ip route get 8.8.8.8 | grep 8.8.8.8 | awk '{ print $7 }'`,IP:127.0.0.1 >> extfile.cnf
    echo extendedKeyUsage = serverAuth >> extfile.cnf
    openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf

    # Creating the client public/private key
    openssl genrsa -out key.pem 4096
    openssl req -subj '/CN=client' -new -key key.pem -out client.csr
    echo extendedKeyUsage = clientAuth > extfile-client.cnf
    openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf

    # Remove unnecessary files
    rm -v client.csr server.csr extfile.cnf extfile-client.cnf

    # Change private and public key permissions
    chmod -v 0400 ca-key.pem key.pem server-key.pem
    chmod -v 0444 ca.pem server-cert.pem cert.pem

    # Moving server certs to their permanent location
    sudo mkdir -p /etc/docker/certs
    sudo mv server*.pem /etc/docker/certs
    sudo cp ca.pem /etc/docker/certs    

    # Add system.d override configuration for docker to start the tcp with tls port
    sudo mkdir -p /etc/systemd/system/docker.service.d/
    sudo su -c 'echo "# /etc/systemd/system/docker.service.d/override.conf
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -H fd:// --tlsverify --tlscacert=/etc/docker/certs/ca.pem --tlscert=/etc/docker/certs/server-cert.pem --tlskey=/etc/docker/certs/server-key.pem -H 0.0.0.0:2376" >> /etc/systemd/system/docker.service.d/override.conf'
    sudo systemctl daemon-reload
    sudo systemctl restart docker

    # Test the TLS connection with curl 
    curl https://127.0.0.1:2376/images/json --cert ~/certs/cert.pem --key ~/certs/key.pem --cacert ~/certs/ca.pem

    # Create an archive with the client certs
    tar czvf certs.tgz ca.pem cert.pem key.pem

The achive file `~/certs/certs.tgz` will have to be transfered to your desktop. We will use its content to log into the docker daemon from your desktop.

## Adding Assemblyline specific configuration 

### Assemblyline folders

Because Assemblyline use it's own set of folders inside the core, service-server and UI container, we have to create the same folder structure here so we can run the components in debug mode.

    sudo mkdir -p ~/git

    sudo mkdir -p /etc/assemblyline
    sudo mkdir -p /var/cache/assemblyline
    sudo mkdir -p /var/lib/assemblyline
    sudo mkdir -p /var/log/assemblyline

    sudo chown $USER /etc/assemblyline
    sudo chown $USER /var/cache/assemblyline
    sudo chown $USER /var/lib/assemblyline
    sudo chown $USER /var/log/assemblyline

### Assemblyline dev configuration files

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
      fqdn: `ip route get 8.8.8.8 | grep 8.8.8.8 | awk '{ print $7 }'`.nip.io
    " > /etc/assemblyline/config.yml

**NOTE:** As you can see in the last command we are setting the FQDN to YOUR_IP.nip.io. NIP.IO is a service that will resolv the first part of the domain **YOUR_IP**.nip.io to it's IP value. We use this to fake DNS when there are none. This is especially useful for oAuth because some providers are forbidding redirect urls to IPs. You can also replace the fqdn with your DNS name if you have one.

## Setup Python Virtual Environments

We will make two python virtual environments: 

 - One for the core components 
 - One for services 

 That should be enough to cover most cases. If a service as conflicting dependancies with other, I suggest you create a seperate virtualenv for it when you try to debug it. The core components should all be fine in the same environment.

### Setting up Core Virtualenv

    # Make sure venv dir exist and we are in it 
    mkdir -p ~/venv
    cd ~/venv

    # Create the virtualenv and activate it
    python3.7 -m venv core
    source ~/venv/core/bin/activate

    # Install assemblyline packages with their test dependencies
    pip install assemblyline[test] assemblyline-core[test] assemblyline-service-server[test] assemblyline-ui[test]

    # Remove assemblyline packages because we will use the live code
    pip uninstall -y assemblyline assemblyline-core assemblyline-service-server assemblyline-ui

### Setting up Service Virtualenv

    # Make sure venv dir exist and we are in it 
    mkdir -p ~/venv
    cd ~/venv

    # Create the virtualenv and activate it
    python3.7 -m venv services
    source ~/venv/services/bin/activate

    # Install assemblyline python client
    pip install assemblyline-client --pre

    # Install assemblyline service packages
    pip install assemblyline-service-client assemblyline-v4-service 

    # Remove assemblyline packages because we will use the live code
    pip uninstall -y assemblyline assemblyline-core assemblyline-service-client assemblyline-v4-service
    
## On your desktop

We are now done setting up the target VM. For the rest of the instructions, we will mainly setup your pycharm IDE to interface with the target VM.


### Get your docker certs and install them 

    mkdir -p ~/docker_certs
    cd ~/docker_certs
    scp USER_OF_TARGET_VM@IP_OF_TARGET_VM:certs/certs.tgz ~
    tar zxvf certs.tgz
    rm certs.tgz

### Install Pycharm 

You can download Pycharm Profesional directly from [jetbrain](https://www.jetbrains.com/pycharm/)'s website but if your desktop is runnning Ubuntu 18.04, you can just install it with snap:

    sudo snap install --classic pycharm-professional

### Install git

You can get git directly from [GIT](https://git-scm.com/downloads)'s website but if your desktop is running Unbuntu 18.04 you can just install it with APT:

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

### Setup Pycharm 

#### Load Alv4 Project

 1. Load **pycharm-professional**
    - Choose whatever configuration option you want until the **Welcome screen**
 2. Click the **Open** button
 3. Choose the **~/git/alv4 directory**

#### Setup remote deployment and interpreter

 1. Click **Files** -> **Settings**
 2. Select **Project: alv4** -> **Python Interpreter**
 3. Click the **cog wheel** on the top right -> **Add** 
 4. Select **SSH Interpreter** -> **New Configuration**
    - Host: IP or DNS name of your target VM
    - Username: username of the user on the target VM
    - Port: 22 unless you changed it...
 5. Click **Next**
 6. Put your target VM password in the box, check **Save password** and click **Next**
 7. In the next window, do the following: 
    - For the **interpreter box**, click the little **folder** and select your core venv (**/home/YOUR_TARGET_USER/venv/core/bin/python**)
    - For the **Sync folders** box, click the little **folder** and for the remote path set the path to: **/home/YOUR_TARGET_USER/git/alv4** then click **OK**
    - Make sure **Automatically upload files to the server** is checked
    - Hit **Finish**
 8. Click **Ok**
 9. Let it load the interpreter and do the transfers

#### Setup project structure

 1. Click **Files** -> **Settings**
 2. Select **Project: alv4** -> **Python Structure**
 3. For each top level folder (**assemblyline-base**, **assemblyline-core**...)
    - Click on it
    - Click **Source**
 4. Select **assemblyline-ui/assemblyline_ui/static/ng-template** then click **Template**
 5. Select **assemblyline-ui/assemblyline_ui/static/** then click **Resources**
 6. Click **apply** then **OK**

#### Setup link to remote docker

 1. Click **Files** -> **Settings**
 2. Select **build, Execution, Deployment** -> **Docker**
 3. Click the little **+** on top left
 4. Select **TCP Socket**
    - In engine API URL put: **https://TARGET_VM_IP:2376**
    - In Certificates folder, click the little folder and browse to **~/docker_certs** directory
 5. Click **OK**
