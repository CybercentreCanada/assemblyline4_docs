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

Run setup script

=== "Core development only"
    ```shell
    cd alv4
    ./setup_vscode.sh
    ```

=== "Core and Services development"
    ```shell
    cd alv4
    ./setup_vscode.sh -s
    ```

## Post-installation instructions

When installation is complete, VSCode should launch in the development workspace. To take full advantage of this setup, we strongly advise installing the recommended extensions when prompted or typing '@recommended' in the Extensions tab.

You can now refer to the [use VSCode](../use_vscode) instructions to get you started using the VSCode environement with Assemblyline.
