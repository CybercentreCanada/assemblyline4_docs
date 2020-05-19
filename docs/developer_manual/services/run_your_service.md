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

To test an AssemblyLine service in standalone mode, the
`run_service_once` module from the [assemblyline-v4-service](https://pypi.org/project/assemblyline-v4-service/) package
can be used to run a single task through the service for testing.

## Setting up dev environment
**NOTE:** The following environment setup has only been tested on Ubuntu 18.04.

1. Install required packages

    ```bash
    sudo apt-get install build-essential libffi-dev python3.7 python3.7-dev python3-pip automake autoconf libtool libfuzzy-dev
    ```
    
2. Install AssemblyLine v4 service package

    ```bash
    pip3 install --user assemblyline-v4-service
    ```
    **NOTE** Ubuntu 18.04 defaults to python3.6 may need to run this instead 
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

## Get your service working inside a container
### Build the service container
1. Change working directory to root of the service:

    ```bash
   cd <path to the tutorial service directory>
   ```
   
2. Run the `docker build` command and tag the container with the same name that container name has in your service manifest

    ```bash
    docker build -t testing/assemblyline-service-resultsample .
   ```

### Add the container to your deployment

1. Using your web browser, go to the service management page: https://localhost/admin/services.html
2. Click the `Add service` button
3. Paste the entire content of the `service_manifest.yaml` file in the text box
4. Click the `Add` button

Your service information has been added to the system. The scaler component should automatically start a container of your newly created service.


