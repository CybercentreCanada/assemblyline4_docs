# Getting Started

Before starting to develop for Assemblyline, you will need to set up your environment. We have a couple of options to assist you, from a free-to-use and easy-to-setup script to more complex setups.

## Development virtual machine

Whether you are developing a new service or working on core components, Assemblyline requires specific external packages and files installed in specific directories in the containers. For this reason, it is recommended that you **do not** develop directly from your desktop unless you use remote debugging features.

### The minimum specifications for development virtual machine

There are quite a few containers to run to spin-up the Assemblyline dependencies. For this reason, the development VM should have at least the following specifications:

- 2 cores
- 8 GB of RAM
- 40 GB of disk space

### Operating system

We recommend that you use [Ubuntu](https://releases.ubuntu.com/) for your development VM, because all instructions that we provide in the document has been built with this OS in mind. There are two versions that you can pick from:

-   \[Recommended\] [Ubuntu 24.04 Desktop](https://releases.ubuntu.com/24.04.3/ubuntu-24.04.3-desktop-amd64.iso)
    - For local development where your IDE runs in the same VM as the Assemblyline containers
-   [Ubuntu 24.04 Server](https://releases.ubuntu.com/24.04.3/ubuntu-24.04.3-live-server-amd64.iso)
    - For remote development where your IDE runs on your local computer and the Assemblyline containers run on the VM

## Choosing your IDE

Many IDEs will give you the features needed to do development. We have seen setups using a combination of terminal tools, PyCharm, and others. We documented a more complete setup using VSCode which support local and remote development:

### VSCode

This is our recommended IDE since it is free and very easy to setup. Most of the Assemblyline team has moved to this IDE and we use a mix of local and remote development with it. Once you're done installing your VM, you can follow the instructions to get VSCode up and running using our simple [setup script](../vscode/setup_script).

For instructions on how to use VSCode, refer to the "[use VSCode](../vscode/use_vscode)" documentation.
