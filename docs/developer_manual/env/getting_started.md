# Getting Started

Before starting to develop for Assemblyline, you will need to setup your environment. We have a couple options for you to choose from free to use and easy to setup script to more complex setups.

## Development Virtual Machine

Whether if you are developping a new service or working on core components, Assemblyline require specific external packages and files installed in specific directories in the containers. For this reason, it is recommended that you **do not** develop directly from your desktop unless you use remote debugging features.

### Virtual Machine Minimum Specs

There are quite a few containers to run to spin-up the Assemblyline dependencies. For this reason, the development VM should have at least the following specs:

 - 2 Cores
 - 8 GB of RAM
 - 40 GB of disk space

### Operating system

We recommend that you use Ubuntu 20.04 for your development VM, because all the instructions that we will provide in the document has been built with this OS in mind. There are two version you can pick from:

* [Ubuntu 20.04 Desktop](https://releases.ubuntu.com/20.04.3/ubuntu-20.04.3-desktop-amd64.iso) - For local development where your IDE runs in the same VM as the Assemblyline containers (Definitely the easiest setup)
* [Ubuntu 20.04 Server](https://releases.ubuntu.com/20.04.3/ubuntu-20.04.3-live-server-amd64.iso) - For remote development where your IDE runs on your local computer and the Assemblyline containers run on the VM

## Choosing your IDE

We have two IDEs for you to pick from, both of which support local and remote development:

### VSCode

This is our recommended IDE since it is free and very easy to setup. Most of Assemblyline team has moved to this IDE and we use a mix of local and remote development with it. Once your done installing your VM, you can follow the instructions to get VSCode up and running using our simple [setup script](../vscode/setup_script).

For instruction on how to use VSCode, refer to the [use VSCode](../vscode/use_vscode) documentation.

### Pycharm

This IDE is much more robust in terms python development but the setup is a lot harder to get right too. There is both a paid and free alternative of this IDE. The free community edition of pycharm will allow you to do local develpment only and you will have to run the dependencies by hand using docker-compose. Were as the paid version, pycharm professional will let you manage docker dependencies inside the IDE and do remote development.

Here are the installation instructions for the different setups:

* [Local development](../pycharm/local_development) (minimum requirement: Pycharm community edition)
* [Remote development](../pycharm/remote_development) (minimum requirement: Pycharm Professional edition)

For instruction on how to use Pycharm, refer to the [use Pycharm](../pycharm/use_pycharm) documentation.
