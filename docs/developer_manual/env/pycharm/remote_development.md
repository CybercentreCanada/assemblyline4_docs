# Remote development

!!! warning
    To use this setup, we assume that you have a paid version of PyCharm ([Pycharm professional](https://www.jetbrains.com/lp/pycharm-pro/)) because we will be using features that are exclusive to the Professional version. If you don't, use the [Local development](../local_development) setup instead.

This document will show you how to set up your target virtual machine for remote development which means that you will run your IDE on your desktop and run the Assemblyline containers on the remote target VM.

## On the target VM

### Operating system

For this document, we will assume that you are working on a fresh installation of [Ubuntu 20.04 Server](https://releases.ubuntu.com/20.04.3/ubuntu-20.04.3-live-server-amd64.iso).

#### Update target VM

Make sure Ubuntu is running the latest software

```shell
sudo apt update
sudo apt dist-upgrade
```

Reboot if needed

```shell
sudo reboot
```

### Installing pre-requisite software

#### Install SSH Daemon

We need to make sure the remote target has an SSH daemon installed for remote debugging

```shell
sudo apt update
sudo apt install -y ssh
```

#### Install Assemblyline APT dependencies

```shell
sudo apt update
sudo apt-get install -yy libfuzzy2 libmagic1 libldap-2.4-2 libsasl2-2 build-essential libffi-dev libfuzzy-dev libldap2-dev libsasl2-dev libssl-dev
```

#### Install Python 3.9

Assemblyline 4 containers are now all built on Python 3.9 therefore we will install Python 3.9.

```shell
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt-get install -yy python3-venv python3.9 python3.9-dev python3.9-venv libffi7
```

#### Installing Docker and Docker Compose

Installing `docker` and `docker compose` on your machine is necessary for Assemblyline remote development.

1. Follow the install guide provided by the official Docker documentation:
    * [Install Guide for Ubuntu](https://docs.docker.com/engine/install/ubuntu)
    * [Install Guide for RHEL](https://docs.docker.com/engine/install/rhel/)
    * [Install Guide for Other Platforms](https://docs.docker.com/engine/install)

2. Ensure the installation was successful by invoking the commands:
  ```shell
  docker version
  docker compose version
  ```

##### Securing Docker for remote access

We are going to make your Docker server accessible from the internet. To make it secure, we need to enable TLS authentication in the Docker daemon. Anywhere that you see assemblyline.local, you can change that value to your own DNS name. If you're planning on using an IP, you'll have to set a `static IP` to the remote VM because your certificate (cert) will only allow connections to that IP.

```shell
# Create a cert directory
mkdir ~/certs
cd ~/certs

# Create a CA (Remember the password you've set)
openssl genrsa -aes256 -out ca-key.pem 4096

# Create a certificate-signing request (ignore the .rng error)
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

# Add system.d override configuration for Docker to start the TCP with TLS port
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
```

The archive file `~/certs/certs.tgz` will have to be transferred to your desktop. We will use its content to log into the Docker daemon from your desktop.

### Adding Assemblyline specific configuration

#### Assemblyline folders

Because Assemblyline uses its own set of folders inside the core, service-server, and UI container, we have to create the same folder structure here so that we can run the components in debug mode.

```shell
sudo mkdir -p ~/git

sudo mkdir -p /etc/assemblyline
sudo mkdir -p /var/cache/assemblyline
sudo mkdir -p /var/lib/assemblyline
sudo mkdir -p /var/log/assemblyline

sudo chown $USER /etc/assemblyline
sudo chown $USER /var/cache/assemblyline
sudo chown $USER /var/lib/assemblyline
sudo chown $USER /var/log/assemblyline
```

#### Assemblyline dev configuration files

Here we will create configuration files that match the default dev Docker Compose configuration files so that we can swap any of the components to the one that is being debugged.

```shell
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
```

!!! tip
    As you can see in the last command we are setting the FQDN to YOUR_IP.nip.io. NIP.IO is a service that will resolve the first part of the domain `YOUR_IP`.nip.io to its IP value. We use this to fake DNS when there are none. This is especially useful for oAuth because some providers are forbidding redirect URLs to IPs. You can also replace the FQDN with your own DNS name if you have one.

### Setup Python Virtual Environments

We will make two Python virtual environments:

- One for the core components
- One for services

That should be enough to cover most cases. If a service has conflicting dependencies with another, I suggest you create a separate virtualenv for it when you try to debug it. The core components should all be fine in the same environment.

#### Setting up Core Virtualenv

```shell
# Make sure the venv directory exists and we are in it
mkdir -p ~/venv
cd ~/venv

# Create the virtualenv
python3.9 -m venv core

# Install Assemblyline packages with their test dependencies
~/venv/core/bin/pip install assemblyline[test] assemblyline-core[test] assemblyline-service-server[test] assemblyline-ui[test]

# Remove Assemblyline packages because we will use the live code
~/venv/core/bin/pip uninstall -y assemblyline assemblyline-core assemblyline-service-server assemblyline-ui
```

#### Setting up Service Virtualenv (optional)

```shell
# Make sure the venv directory exists and we are in it
mkdir -p ~/venv
cd ~/venv

# Create the virtualenv
python3.9 -m venv services

# Install Assemblyline Python client
~/venv/services/bin/pip install assemblyline-client

# Install Assemblyline service packages
~/venv/services/bin/pip install assemblyline-service-client assemblyline-v4-service

# Remove Assemblyline packages because we will use the live code
~/venv/services/bin/pip uninstall -y assemblyline assemblyline-core assemblyline-service-client assemblyline-v4-service
```

## On your desktop

We are now done setting up the target VM. For the rest of the instructions, we will mainly setup your PyCharm IDE to interface with the target VM.

### Get your Docker certs and install them

```shell
mkdir -p ~/docker_certs
cd ~/docker_certs
scp USER_OF_TARGET_VM@IP_OF_TARGET_VM:certs/certs.tgz ~/docker_certs/
tar zxvf certs.tgz
rm certs.tgz
```

### Install PyCharm

You can download PyCharm Professional directly from [JetBrains](https://www.jetbrains.com/pycharm/)'s website but if your desktop is running Ubuntu 20.04, you can just install it with `snap`:

```shell
sudo snap install --classic pycharm-professional
```

### Install Git

You can get Git directly from [GIT](https://git-scm.com/downloads)'s website but if your desktop is running Ubuntu 20.04 you can just install it with `APT`:

```shell
sudo apt install -y git
```

!!! tip
    You should add your desktop SSH keys to your GitHub account to use Git via SSH. Follow these instructions to do so: [GitHub Help](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

### Clone repositories

#### Core components

Create the core working directory

```shell
mkdir -p ~/git/alv4
cd ~/git/alv4
```

Clone Assemblyline's repositories

=== "Git via SSH"
    Use SSH if you have your SSH id_rsa file configured to your GitHub account
    ```shell
    git clone git@github.com:CybercentreCanada/assemblyline-base.git
    git clone git@github.com:CybercentreCanada/assemblyline-core.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-client.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-server.git
    git clone git@github.com:CybercentreCanada/assemblyline-ui.git
    git clone git@github.com:CybercentreCanada/assemblyline-v4-service.git
    ```

=== "Git via HTTPS"
    Use HTTPS if you don't have your GitHub account configured with an SSH key
    ```shell
    git clone https://github.com/CybercentreCanada/assemblyline-base.git
    git clone https://github.com/CybercentreCanada/assemblyline-core.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-client.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-server.git
    git clone https://github.com/CybercentreCanada/assemblyline-ui.git
    git clone https://github.com/CybercentreCanada/assemblyline-v4-service.git
    ```

#### Services (optional)

Create the service working directory

```shell
mkdir -p ~/git/services
cd ~/git/services
```

Clone Assemblyline's services repositories
=== "Git via SSH"
    Use SSH if you have your SSH id_rsa file configured to your GitHub account
    ```shell
    git clone git@github.com:CybercentreCanada/assemblyline-service-apivector.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-antivirus.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-apkaye.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-avclass.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-batchdeobfuscator.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-capa.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-cape.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-characterize.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-configextractor.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-deobfuscripter.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-elf.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-elfparser.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-emlparser.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-espresso.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-extract.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-floss.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-frankenstrings.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-iparse.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-metapeek.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-oletools.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-pdfid.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-peepdf.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-pe.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-pixaxe.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-safelist.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-sigma.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-suricata.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-swiffer.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-torrentslicer.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-unpacker.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-unpacme.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-vipermonkey.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-virustotal.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-yara.git
    ```

=== "Git via HTTPS"
    Use HTTPS if you don't have your GitHub account configured with an SSH key
    ```shell
    git clone https://github.com/CybercentreCanada/assemblyline-service-apivector.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-antivirus.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-apkaye.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-avclass.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-batchdeobfuscator.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-capa.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-cape.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-characterize.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-configextractor.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-elf.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-elfparser.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-emlparser.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-espresso.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-extract.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-floss.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-frankenstrings.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-iparse.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-metapeek.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-oletools.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-pdfid.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-peepdf.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-pe.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-pixaxe.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-safelist.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-sigma.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-suricata.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-swiffer.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-torrentslicer.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-unpacker.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-unpacme.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-vipermonkey.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-virustotal.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-yara.git
    ```

### Setup PyCharm for core

Start with loading the core directory in Pycharm:

!!! example "Load core folder"
    1. Load Pycharm Professional
        - Choose whatever configuration option you want until the `Welcome screen`
    2. Click the `Open` button
    3. Choose the `~/git/alv4` directory

The setup of the remote deployment interpreter:

!!! example "Setup core remote interpreter"
    1. Click `Files` -> `Settings`
    2. Select `Project: alv4` -> `Python Interpreter`
    3. Click the `cog wheel` on the top right -> `Add`
    4. Select `SSH Interpreter` -> `New Configuration`
        - Host: IP or DNS name of your target VM
        - Username: username of the user on the target VM
        - Port: 22 unless you changed it...
    5. Click `Next`
    6. Put your target VM password in the box, check `Save password`, and click `Next`
    7. In the next window, do the following:
        - For the `interpreter box`, click the little `folder` and select your core venv (`/home/YOUR_TARGET_USER/venv/core/bin/python3.9`)
        - For the `Sync folders` box, click the little `folder` and for the remote path set the path to: `/home/YOUR_TARGET_USER/git/alv4` then click `OK` (ensure target directory has write permissions for all users)
        - Make sure `Automatically upload files to the server` is checked
        - Make sure  `Execute code using this interpreter with root privileges via sudo` is checked
        - Hit `Finish`
    8. Click `Ok`
    9. Let it load the interpreter and do the transfers

Finally link Docker for remote management:

!!! example "Setup Docker remote management"
    1. Click `Files` -> `Settings`
    2. Select `build, Execution, Deployment` -> `Docker`
    3. Click the little `+` on top left
    4. Select `TCP Socket`
        - In engine API URL put: `https://TARGET_VM_IP:2376`
        - In Certificates folder, click the little folder and browse to `~/docker_certs` directory
    5. Click `OK`

### Setup PyCharm for service (optional)

Start with loading the core directory in PyCharm:

!!! example "Load services folder"
    1. From your core PyCharm window open the `File menu` then click `Open`
    3. Choose the `~/git/services` directory
    3. Select `New Window`

The setup of the remote deployment interpreter:

!!! example "Setup services remote interpreter"
    1. Click `Files` -> `Settings`
    2. Select `Project: services` -> `Python Interpreter`
    3. Click the `cog wheel` on the top right -> `Add`
    4. Select `SSH Interpreter` -> `New Configuration`
        - Host: IP or DNS name of your target VM
        - Username: username of the user on the target VM
        - Port: 22 unless you changed it...
    5. Click `Next`
    6. Put your target VM password in the box, check `Save password`, and click `Next`
    7. In the next window, do the following:
        - For the `interpreter box`, click the little `folder` and select your core venv (`/home/YOUR_TARGET_USER/venv/services/bin/python3.9`)
        - For the `Sync folders` box, click the little `folder` and for the remote path set the path to: `/home/YOUR_TARGET_USER/git/services` then click `OK` (ensure target directory has write permissions for all users)
        - Make sure `Automatically upload files to the server` is checked
        - Make sure  `Execute code using this interpreter with root privileges via sudo` is checked
        - Hit `Finish`
    8. Click `Ok`
    9. Let it load the interpreter and do the transfers

## Use Pycharm

Now that your remote development VM is set up you should read the [use PyCharm](../use_pycharm) documentation to get yourself started.
