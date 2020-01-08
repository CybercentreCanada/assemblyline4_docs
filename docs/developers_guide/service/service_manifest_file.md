---
layout: default
title: Service manifest file
parent: Service
grand_parent: Developer's guide
nav_order: 3
---

# Service manifest overview
Every service must have a `service_manifest.yml` file in its root directory. The manifest file presents essential
information about the service to the Assemblyline core system, information the Assemblyline core system must have before
it can run the service. 

The diagram below shows all the elements that the manifest file can contain, including a brief description of each.

```yaml
# Name of the service
name: ResultSample
# Version of the service
version: 1
# Description of the service
description: >
  ALv4 Result example service

  This service provides examples of how to:
     - define your service manifest
     - use the different section types
     - use tags
     - use heuristics to score sections
     - use the att&ck matrix
     - use the updater framework
     - define submission parameters
     - define service configuration parameters

# Regex defining the types of files the service accepts and rejects
accepts: .*
rejects: empty|metadata/.*

# At which stage the service should run (one of: FILTER, EXTRACT, CORE, SECONDARY, POST)
# NOTE: Stages are executed in the order defined in the list
stage: CORE
# Which category the service is part of (one of: Antivirus, Dynamic Analysis, External, Extraction, Filtering, Networking, Static Analysis)
category: Static Analysis

# Does the service require access to the file to perform its task
# If set to false, the service will only have access to the file metadata (e.g. Hashes, size, type, ...)
file_required: true
# Maximum execution time the service has before it's considered to be timed out
timeout: 60
# Does the service force the caching of results to be disabled
# (only use for service that will always provided different results each run)
disable_cache: false

# is the service enabled by default
enabled: true
# does the service make APIs call to other product not part of the assemblyline infrastructure (e.g. VirusTotal, ...)
is_external: false
# Number of concurrent services allowed to run at the same time
licence_count: 0

# service configuration block (dictionary of config variables)
# NOTE: The key names can be anything and the value can be of any types
config:
  str_config: value1
  int_config: 1
  list_config: [1, 2, 3, 4]
  bool_config: false

# submission params block: a list of submission param object that define parameters
#                          that the user can change about the service for each of its scans
# supported types: bool, int, str, list
submission_params:
  - default: ""
    name: password
    type: str
    value: ""
  - default: false
    name: extra_work
    type: bool
    value: false

# Service heuristic blocks: List of heuristics object that define the different heuristics used in the service
heuristics:
  - description: This the first Heuristic for ResultSample service.
    filetype: pdf
    heur_id: AL_RESULTSAMPLE_1
    name: Masks has PDF
    score: 100
    attack_id: T1001
  - description: This is second Heuristic for ResultSample service.
    filetype: exe
    heur_id: AL_RESULTSAMPLE_2
    name: Drops an exe
    score: 1000
  - description: This is third Heuristic for ResultSample service.
    filetype: exe
    heur_id: AL_RESULTSAMPLE_3
    name: Extraction information
    score: 0

# Docker configuration block which defines:
#  - the name of the docker container that will be created
#  - cpu and ram allocation by the container
docker_config:
  image: cccs/assemblyline-service-resultsample:latest
  cpu_cores: 1.0
  ram_mb: 1024

# Update configuration block
update_config:
  # update method either run or build
  # run = run the provided update container
  # build = build the provided dockefile
  method: run
  # list of source object from where to fetch files for update and what will be the name of those files on disk
  sources:
    - uri: https://file-examples.com/wp-content/uploads/2017/02/zip_2MB.zip
      name: sample_2mb_file
    - uri: https://file-examples.com/wp-content/uploads/2017/02/zip_5MB.zip
      name: sample_5mb_file
  # intervale in seconds at which the updater runs
  update_interval_seconds: 300
  # Should the downloaded files be used to create signatures in the system
  generates_signatures: false
  # options provided to the build or run command
  run_options:
    image: cccs/assemblyline_dev:latest
    command: python3 -m assemblyline_core.updater.url_update
```
