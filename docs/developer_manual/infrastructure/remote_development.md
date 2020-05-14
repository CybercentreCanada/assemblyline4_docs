---
layout: default
title: Remote development
parent: Infrastructure
grand_parent: Developer's manual
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

    sudo mkdir -p /etc/assemblyline
    sudo mkdir -p /var/cache/assemblyline
    sudo mkdir -p /var/lib/assemblyline
    sudo mkdir -p /var/log/assemblyline

    sudo chown $USER /etc/assemblyline
    sudo chown $USER /var/cache/assemblyline
    sudo chown $USER /var/lib/assemblyline
    sudo chown $USER /var/log/assemblyline

### Assemblyline dev configuration files

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