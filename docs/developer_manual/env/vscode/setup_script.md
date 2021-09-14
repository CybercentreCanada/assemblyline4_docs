# Setup script

Assemblyline's VSCode installation is now entirely scripted. It will setup for you the following things for you:

- Install VSCode via snap
- Install AL4 development dependencies
- Create a virtual python environment for your project
- Create Run targets inside VSCode for all core components and other important scripts
- Create Tasks inside VSCode for development using Docker-Compose
- Setup our code formatting standards
- Deploy a local Docker registry on port 32000

We recommend installing the VSCode extensions needed to use this environment once VSCode is launched in the workspace.

## Pre-requisites

The setup script assumes the following:

- You are running this on a Ubuntu machine / VM (20.04 and up)
- VSCode does not have to be running on the same host were you run this script so run this setup script on the target VM of a remote development setup
- You have read the setup_vscode.sh script, this script will install and configure packages for ease of use.

!!! tip
    If you are uncomfortable which some of the changes that it makes, you should comment them before running the script.

## Installation instruction

Create your repo directory
```shell
mkdir -p ~/git
cd ~/git
```

Clone repo
```shell
git clone https://github.com/CybercentreCanada/assemblyline-development-setup alv4
```

Run setup script. Choose the type of development you want to do and on what type of system.

=== "Core only (Local)"
    ```shell
    cd alv4
    ./setup_vscode.sh -c
    ```

=== "Core and Services (Local)"
    ```shell
    cd alv4
    ./setup_vscode.sh -c -s
    ```

=== "Core only (Remote)"
    ```shell
    cd alv4
    ./setup_vscode.sh
    ```

=== "Core and Services (Remote)"
    ```shell
    cd alv4
    ./setup_vscode.sh -s
    ```

!!! important
    When running the setup script with the `Core and Services` you will get two dev folder: `~/git/alv4` and `~/git/services`.

    The reason for this is that we want to make sure that services python dependencies don't interfere with core services dependencies. Therefor the `venv` are create with a different set of dependencies. The service `venv` will point to the core live code to install `assemblyline-base`, `assemblyline-core`, `assemblyline-v4-service` and `assemblyline-client`. That way, any modification you do to core packages it code will be reflected in your service instantly.

## Post-installation instructions

When installation is complete, you will be asked to reboot the VM. This is required for sudoless docker to work.

After the VM has done rebooted, you can use a shell to open VSCode:

```shell
code ~/git/alv4
```

!!! note
    If you've installed the services, you should open another code window pointing to the services folder.
    ```shell
    code ~/git/services
    ```

### Install recommended extensions

To take full advantage of this setup, we strongly advise installing the recommended extensions when prompted or typing `@recommended` in the `Extensions tab`.

![Recommended extensions](./images/recommended.png)

### Start using VSCode

You can now refer to the [use VSCode](../use_vscode) instructions to get you started using the VSCode environment with Assemblyline.
