# Local development

This documentation will show you how to set up your development virtual machine for local development using the PyCharm Community Edition IDE (This would also work with PyCharm Professional). In this setup, you will run your IDE inside the virtual machine where the Assemblyline containers are running. This is by far the easiest setup to get PyCharm working and removes a lot of headaches.

## Operating system

For this documentation, we will assume that you are working on a fresh installation of [Ubuntu 20.04 Desktop](https://releases.ubuntu.com/20.04.3/ubuntu-20.04.3-desktop-amd64.iso).

### Update VM

Make sure Ubuntu is running the latest software
```shell
sudo apt update
sudo apt dist-upgrade
sudo snap refresh
```

Reboot if needed
```shell
sudo reboot
```

## Installing pre-requisite software

### Install Assemblyline APT dependencies

```shell
sudo apt update
sudo apt-get install -yy libfuzzy2 libmagic1 libldap-2.4-2 libsasl2-2 build-essential libffi-dev libfuzzy-dev libldap2-dev libsasl2-dev libssl-dev
```

### Install Python 3.9

Assemblyline 4 containers are now all built on Python 3.9 therefore we will install Python 3.9.

```shell
sudo apt install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt-get install -yy python3-venv python3.9 python3.9-dev python3.9-venv libffi7
```

### Installing Docker

Follow these simple commands to get Docker running on your machine:
```shell
# Add Docker repository
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Test Docker installation
sudo docker run hello-world
```
### Installing docker-compose

Installing docker-compose is done the same way on all Linux distros. Follow these simple instructions:
```shell
# Install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Test docker-compose installation
docker-compose --version
```

### Installing PyCharm

Let's install the desired PyCharm version. The Professional version is not needed but if you have a licence you can use it.

=== "PyCharm Community"

    ```shell
    sudo snap install --classic pycharm-community
    ```

=== "PyCharm Professional"

    ```shell
    sudo snap install --classic pycharm-professional
    ```

## Adding Assemblyline specific configuration

### Data directories

Because Assemblyline uses its own set of folders inside the core, service-server, and UI containers, we must create the same folder structure here so we can run the components in debug mode.

```shell
sudo mkdir -p /etc/assemblyline
sudo mkdir -p /var/cache/assemblyline
sudo mkdir -p /var/lib/assemblyline
sudo mkdir -p /var/log/assemblyline

sudo chown $USER /etc/assemblyline
sudo chown $USER /var/cache/assemblyline
sudo chown $USER /var/lib/assemblyline
sudo chown $USER /var/log/assemblyline
```

### Dev default configuration files

Here we will create configuration files that match the default dev docker-compose configuration files so that we can swap any of the components to the one that is being debugged.

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
  fqdn: 127.0.0.1.nip.io
" > /etc/assemblyline/config.yml
```

!!! tip
    As you can see in the last command we are setting the FQDN to 127.0.0.1.nip.io. NIP.IO is a service that will resolve the first part of the domain **127.0.0.1**.nip.io to its IP value. We use this to fake DNS when there are none. This is especially useful for oAuth because some providers are forbidding redirect URLs to IPs.

## Setup Assemblyline source

### Install git

Since your VM is running Ubuntu 20.04 you can just install it with APT:

```shell
sudo apt install -y git
```

!!! tip
    You should add your desktop SSH keys to your GitHub account to use Git via SSH. Follow these instructions to do so: [GitHub Help](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

### Clone core repositories

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

### Virtual Environment

```shell
# Directly in the alv4 source directory
cd ~/git/alv4

# Create the virtualenv
python3.9 -m venv venv

# Install Assemblyline packages with their test dependencies
~/git/alv4/venv/bin/pip install assemblyline[test] assemblyline-core[test] assemblyline-service-server[test] assemblyline-ui[test]

# Remove Assemblyline packages because we will use the live code
~/git/alv4/venv/bin/pip uninstall -y assemblyline assemblyline-core assemblyline-service-server assemblyline-ui
```

## Setting up Services (Optional)
If you plan on doing service development in PyCharm you will need a dedicated directory for services with its own virtual environment.

### Clone service repositories

Create the service working directory
```shell
mkdir -p ~/git/services
cd ~/git/services
```

Clone Assemblyline's services repositories
=== "Git via SSH"
    Use SSH if you have your SSH id_rsa file configured to your GitHub account
    ```shell
    git clone git@github.com:CybercentreCanada/assemblyline-service-antivirus.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-apkaye.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-avclass.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-beaver.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-characterize.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-configextractor.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-cuckoo.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-deobfuscripter.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-emlparser.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-espresso.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-extract.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-floss.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-frankenstrings.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-iparse.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-metadefender.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-metapeek.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-oletools.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-pdfid.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-peepdf.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-pefile.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-pixaxe.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-safelist.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-sigma.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-suricata.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-swiffer.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-torrentslicer.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-unpacker.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-unpacme.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-vipermonkey.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-virustotal-dynamic.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-virustotal-static.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator.git
    git clone git@github.com:CybercentreCanada/assemblyline-service-yara.git
    ```

=== "Git via HTTPS"
    Use HTTPS if you don't have your GitHub account configured with an SSH key
    ```shell
    git clone https://github.com/CybercentreCanada/assemblyline-service-antivirus.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-apkaye.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-avclass.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-beaver.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-characterize.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-configextractor.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-cuckoo.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-emlparser.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-espresso.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-extract.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-floss.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-frankenstrings.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-iparse.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-metadefender.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-metapeek.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-oletools.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-pdfid.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-peepdf.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-pefile.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-pixaxe.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-safelist.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-sigma.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-suricata.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-swiffer.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-torrentslicer.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-unpacker.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-unpacme.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-vipermonkey.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-virustotal-dynamic.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-virustotal-static.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator.git
    git clone https://github.com/CybercentreCanada/assemblyline-service-yara.git
    ```

### Virtual Environment

```shell
# Directly in the services source directory
cd ~/git/services

# Create the virtualenv
python3.9 -m venv venv

# Install Assemblyline packages from source in the services virtualenv
~/git/services/venv/bin/pip install -e ~/git/alv4/assemblyline-base
~/git/services/venv/bin/pip install -e ~/git/alv4/assemblyline-core
~/git/services/venv/bin/pip install -e ~/git/alv4/assemblyline-service-client
~/git/services/venv/bin/pip install -e ~/git/alv4/assemblyline-v4-service
```

## Setup PyCharm for core

1. Load PyCharm
    - Choose whatever configuration option you want until the `Welcome screen`
2. Click the `Open` button
3. Choose the `~/git/alv4` directory

!!! info
    Your Python interpreter shows as `No Interpreter` in the bottom right corner of the window, do the following:

    1. Click on it
    2. Click add interpreter
    3. Choose "Existing Environment"
    4. Click "OK"

## Setup PyCharm for service (optional)

1. From your core PyCharm window open the `File menu` then click `Open`
2. Choose the `~/git/services` directory
3. Select `New Window`

!!! info
    Your Python interpreter shows as `No Interpreter` in the bottom right corner of the window, do the following:

    1. Click on it
    2. Click add interpreter
    3. Choose "Existing Environment"
    4. Click "OK"

## Use PyCharm

Now that your Local development VM is set up you should read the [use PyCharm](../use_pycharm) documentation to get you started.
