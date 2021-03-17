---
layout: default
title: Run your new service
parent: Services
grand_parent: Developer's manual
nav_order: 2
has_toc: false
---

# Run your service
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

To test an Assemblyline service in standalone mode, the
`run_service_once` module from the [assemblyline-v4-service](https://pypi.org/project/assemblyline-v4-service/) package
can be used to run a single task through the service for testing.

## Setting up dev environment
**NOTE:** The following environment setup has only been tested on Ubuntu 18.04.

1. Install required packages

    ```bash
    sudo apt-get install build-essential libffi-dev python3.7 python3.7-dev python3-pip automake autoconf libtool libfuzzy-dev
    ```
    
2. Install Assemblyline 4 service package

    ```bash
    pip3 install --user assemblyline-v4-service
    ```
    **NOTE** Ubuntu 18.04 defaults to Python3.6 may need to run this instead 
    ```bash
    python3.7 -m pip install --user assemblyline-v4-service
    ```
3. Add your service development directory path (ie. `/home/ubuntu/alv4_services`) to the PYTHONPATH environment variable

## Using the *run_service_once* module
### Steps
1. Ensure the current working directory is the root of the service directory of the service to be run

    ```bash
    cd <service root directory>
   ```
   
2. From a terminal, run the `run_service_once` module, where `<service path>` is the path to the service module and `<file path>` is the path of the file to be processed

    ```bash
   python3.7 -m assemblyline_v4_service.dev.run_service_once <service path> <file path>
   ```
   
3. The output of the service (`result.json` and extracted/supplementary files) will be located in a directory where the
   input file is located 
   
### Example of running the Tutorial service
1. Change working directory to root of the service:

    ```bash
   cd <path to the tutorial service directory>
   ```
   
2. Run the `run_service_once` module

    ```bash
    python3.7 -m assemblyline_v4_service.dev.run_service_once result_sample.ResultSample <path to a file to scan>
   ```
   
3. The `results.json` and any extracted/supplementary files will be outputted to `<previously provided path to a file to scan>_resultsample` directory

## Using the *run_service* module
### Use Case
The following technique is how to hook in a service to an Assemblyline instance without using a Docker container for the service.
The purpose of this technique is if you wish to avoid Docker container use for your service (if you don't, see 
[Get your service working inside a container](https://cybercentrecanada.github.io/assemblyline4_docs/docs/developer_manual/services/run_your_service.html#get-your-service-working-inside-a-container)).

### Prerequisites
- You will need a virtual machine that is hosting an Assemblyline instance (see [Getting Started](https://cybercentrecanada.github.io/assemblyline4_docs/docs/developer_manual/Assemblyline/getting_started.html)).
and depending on your virtual machine, perform the following setup: [Local development](https://cybercentrecanada.github.io/assemblyline4_docs/docs/developer_manual/Assemblyline/local_development.html) or 
[Remote development](https://cybercentrecanada.github.io/assemblyline4_docs/docs/developer_manual/Assemblyline/remote_development.html).
- You will need to install all packages required by your service in the virtual machine (both pip and apt).
- You will need to upload the repositories of your service and the assemblyline-service-client to the virtual machine.

### Steps for Running in PyCharm
1. Set up the Task Handler
- Right click on `assemblyline-service-client/assemblyline_service_client/task_handler.py` in your project window and select `Create 'task_handler'...`.
- In the window "Create Run Configuration: 'task_handler'" set the following:
  - Name: task_handler
  - Script path: `<your path>/assemblyline-service-client/assemblyline_service_client/task_handler.py`
  - Python interpreter: `<the Python interpreter on the virtual machine>`
  - Working directory: `<your path>/assemblyline-service-client/assemblyline_service_client`
- OK

2. Set up `run_service` for your service
- Run -> Edit Configurations
- In the window "Run/Debug Configurations", set the following:
  - Click the "+"
  - Select "Python"
  - Name: <Your service name> via RunService
  - Module name: `assemblyline_v4_service.run_service`
  - Environment variables: `<default variables (usually PYTHONBUFFERED=1)>;SERVICE_PATH=<module path to main class in service>`
  - Python interpreter: `<the Python interpreter on the virtual machine>`
  - Working directory: `<your path>/assemblyline-service-<service name>`
- OK

3. Run/debug the `task_handler` run configuration
- The task handler awaits a service to be registered...
4. Run/debug the `<Your service name> via RunService` run configuration
- The service registers with the task handler, task handler exits
5. Note that the task handler has now registered your service and exited. So you need to 
run the `task_handler` run configuration once more before you can submit tasks to it via the UI.
6. Use a web browser to head to the service configuration page at https://<virtual machine IP>>/admin/services.html
7 You should see your service loaded and disabled. Enable it and submit away!

Note: If you are making changes to your service and loading it into the Assemblyline instance using this technique, 
you will need to restart the `<Your service name> via RunService` run configuration every time you want your code changes
updated.

## Get your service working inside a container

### Create a private registry
1. The following link will show you how to setup a private Docker image registry: 
[Docker Docs - Deploy a registry server](https://docs.docker.com/registry/deploying/)

2. To test your registry setup, try running the following:
    ```
   curl https://<registry_server_name>/v2/_catalog
    ```
   This should return a list of repositories (likely empty if no images have been pushed yet).
3. Setup the PRIVATE_REGISTRY environment variable in the .env file so that it points to your private registry:
    ```
    services:
      image_variables:
        PRIVATE_REGISTRY: <registry_server_name>/
    ```

### Build/Push the service container to your registry
1. Change working directory to root of the service:

    ```bash
   cd <path to the tutorial service directory>
   ```
   
2. Run the `docker build` command and tag the container with the same name that container name has in your service manifest

    ```bash
    docker build -t <registry_server_name>/assemblyline-service-resultsample .
   ```
3. Push the container image to your private registry

    ```bash
    docker push <registry_server_name>/assemblyline-service-resultsample
   ```

### Add the container to your deployment

1. Using your web browser, go to the service management page: https://localhost/admin/services.html
2. Click the `Add service` button
3. Paste the entire content of the `service_manifest.yaml` file in the text box.

    **Note**: Ensure the service manifest's docker images are prefixed with ${PRIVATE_REGISTRY} to make sure it's pulling from your private repository.
    ```
    docker_config:
      image: ${PRIVATE_REGISTRY}assemblyline-service-resultsample:$SERVICE_TAG
    ```

4. Click the `Add` button

Your service information has been added to the system. The scaler component should automatically start a container of your newly created service.


