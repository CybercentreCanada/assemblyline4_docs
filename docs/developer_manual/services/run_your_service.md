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
[run_service_once.py](https://bitbucket.org/cse-assemblyline/alv4_service/src/master/dev/run_service_once.py) script
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
    
3. Add your service development directory path (ie. `/home/ubuntu/alv4_services`) to the PYTHONPATH environment variable

## Using the *run_service_once* script
### Steps
1. Ensure the current working directory is the root of the service directory of the service to be run

    ```bash
    cd alsvc_<service name>
   ```
   
2. From a terminal, run the `run_service_once` script, where `<service path>` is the path to the service module and `<file path>` is the path of the file to be processed

    ```bash
   python3.7 -m assemblyline_v4_service.dev.run_service_once <service path> <file path>
   ```
   
3. The output of the service (`result.json` and extracted/supplementary files) will be located in a directory where the
   input file is located 
   
### Example of running the ResultSample service
1. Change working directory to root of the service:

    ```bash
   cd assemblyline_result_sample_service
   ```
   
2. From a terminal, run the `run_service_once` script

    ```bash
    python3.7 -m assemblyline_v4_service.dev.run_service_once assemblyline_result_sample_service.result_sample.ResultSample /home/ubuntu/testfile.doc
   ```
   
3. The `results.json` and any extracted/supplementary files will be outputted to `/home/ubuntu/testfile_resultsample`
