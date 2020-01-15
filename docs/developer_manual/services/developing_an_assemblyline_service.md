---
layout: default
title: Developing an AssemblyLine service 
parent: Services
grand_parent: Developer's manual
nav_order: 1
has_toc: false
---

# Developing an AssemblyLine service 
{: .no_toc }

This guide has been created for developers who are looking to develop [services]() for 
AssemblyLine. It is aimed for individuals with general software development knowledge and basic Python skills. In-depth 
knowledge of the AssemblyLine framework is not required to develop a service. 

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Pre-requisites
Before getting started, ensure you have the following setup:
- [AssemblyLine development environment]()
- [Python development environment]() 

## Build you first service
This section will guide you through the bare minimum steps required to create a running, but functionally useless
service. Each sub-section below outlines the steps required for each of the different files required to create an
AssemblyLine service. All files created in the following sub-sections must be placed in a common directory, for 
example `tutorial_service`. 

### 1. Service Python code
Create a Python file named `tutorial_service.py` with the following contents.
```python
from assemblyline_v4_service.common.base import ServiceBase
from assemblyline_v4_service.common.result import Result, ResultSection

class TutorialService(ServiceBase):
   def __init__(self, config=None):
       super(TutorialService, self).__init__(config)

   def start(self):
       self.log.info(f"start() from {self.service_attributes.name} service called")

   def execute(self, request):
       result = Result()
   
       text_section = ResultSection("Example of a default section")           
       text_section.add_line("This line is part of the section body")
       text_section.set_heuristic(1)
       
       result.add_section(text_section)
```

#### *ServiceBase* class
The main class of your AssemblyLine service must inherit from the `ServiceBase` class which can be imported from
`assemblyline_v4_service.common.base`. The `ServiceBase` base class includes many instance variables which can be used
to access service related information. The following tables describes all the available instance variables.

| Variable name | Description |
|:---|:---|
| config | Reference to the service parameters containing values updated by the user for service configuration. |
| log | Reference to the logger. |
| service_attributes | Service attributes from the [service manifest](service_manifest.md). |
| working_directory | The directory which the service can use to temporarily store files during each task execution. |

#### *start* function
The `start` function is called when the AssemblyLine service is initiated and should be used to prepare your service 
for task execution. This function is optional to implement.

#### *execute* function
The `execute` function is called every time your service receives a new file to scan. This is where you should execute 
your processing code. In the example above, we will only generate static results for illustration purposes.

### 2. Service manifest YAML
Create a YAML file named `service_manifest.yml` with the following contents.
```yaml
name: TutorialService
version: 1
description: >
  ALv4 tutorial service

  This service provide is meant to be a basic service example which runs, but is functionally useless.

accepts: .*
rejects: empty|metadata/.*

stage: CORE
category: Static Analysis

file_required: true
timeout: 10
disable_cache: false

enabled: true
is_external: false
licence_count: 0

config:
  str_config: value1
  int_config: 1
  list_config: [1, 2, 3, 4]
  bool_config: false

submission_params:
  - default: ""
    name: password
    type: str
    value: ""
  - default: false
    name: extra_work
    type: bool
    value: false

heuristics:
  - description: This suspicious heuristic fakes making as a PDF
    filetype: "*"
    heur_id: 1
    name: Masks has PDF
    score: 100
  - description: This malicious heuristic fakes dropping and side-loading a DLL and has an Att&ck ID associated with it
    filetype: "*"
    heur_id: 2
    name: Drops an exe
    score: 1000
    attack_id: T1073
  - description: This informational heuristic fakes extracting a configuration block
    filetype: "*"
    heur_id: 3
    name: Extraction config information
    score: 10
  - description: This suspicious heuristic fakes decoding a configuration block and has an Att&ck ID associated with it
    filetype: "*"
    heur_id: 4
    name: Config decoding
    score: 100
    attack_id: T1027

docker_config:
  image: cccs/assemblyline-service-resultsample:latest
  cpu_cores: 1.0
  ram_mb: 1024

update_config:
  method: run
  sources:
    - uri: https://file-examples.com/wp-content/uploads/2017/02/zip_2MB.zip
      name: sample_2mb_file
    - uri: https://file-examples.com/wp-content/uploads/2017/02/zip_5MB.zip
      name: sample_5mb_file
  update_interval_seconds: 300
  generates_signatures: false
  run_options:
    image: cccs/assemblyline_dev:latest
    command: ["python3", "-m", "assemblyline_core.updater.url_update"]
```
A description of all the valid fields in a `service_manifest.yml` are available [here](service_manifest.md).