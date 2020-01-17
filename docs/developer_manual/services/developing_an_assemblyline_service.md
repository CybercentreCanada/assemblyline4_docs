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

### Service Python code
Create a Python file named `tutorial_service.py` with the following contents.
```python
from assemblyline_v4_service.common.base import ServiceBase
from assemblyline_v4_service.common.result import Result, ResultSection

class TutorialService(ServiceBase):
   def __init__(self, config=None):
       super(TutorialService, self).__init__(config)

   def start(self):
       self.log.info(f"start() from {self.service_attributes.name} service called")
   
   def get_tool_version(self):
       return "v1"

   def execute(self, request):
       result = Result()
   
       text_section = ResultSection("Example of a default section")           
       text_section.add_line("This line is part of the section body")
       text_section.set_heuristic(1)
       
       result.add_section(text_section)
   
   def stop(self):
       self.log.info(f"stop() from {self.service_attributes.name} service called")  
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
| working_directory | Returns the directory path which the service can use to temporarily store files during each task execution. |

#### *start* function
The `start` function is called when the AssemblyLine service is initiated and should be used to prepare your service 
for task execution. This function is optional to implement.

#### *get_tool_version* function
The purpose of the `get_tool_version` function is to the return a string of all the tools used by the service. The tool version should be updated to reflect changes in the service tools, so that AssemblyLine can rescan files on the new service version if they are submitted again.

#### *execute* function
The `execute` function is called every time the service receives a new file to scan. This is where you should execute 
your processing code. In the example above, we only generate static results for illustration purposes.

The `request` object is provided as an input to the `execute` function. The `request` object holds information about the
task to be processed by the service. The following table describes all the `request` object variables which the service 
can use. 

| Variable name | Description |
|:---|:---|
| deep_scan | Returns whether the file should be deep scanned or not. |
| file_contents | Returns the raw bytes content of the file to be scanned. |
| file_name | Returns the name of the file (as submitted by the user) to be scanned. |
| file_path | Returns the path to the file to be scanned. The service can use this path directly to access the file. |
| file_type | Returns the AssemblyLine style file type of the file to be scanned. |
| max_extracted | Returns the max number of files that are allowed to be extracted by a service. By default this is set to 500. |
| md5 | Returns the MD5 of the file to be scanned. |
| result | Used to set and get the current result. |
| sha1 | Returns the SHA1 of the file to be scanned. |
| sha256 | Returns the SHA256 of the file to be scanned. |
| temp_submission_data | Can be used to get and set temporary submission data which is passed onto subsequent tasks resulting from adding extracted files. |

The following table describes all the `request` object functions which the service can use.

| Function name | Parameters | Description |
|:---|:---|:---|
| add_extracted | `path`: Complete path to the file <br><br> `name`: Display name of the file <br><br> `description`: Descriptive text about the file <br><br> `classification`: Optional classification of the file | Attach a file extracted by the service to the result. The extracted file will also be scanned through a set of services, as if it had been originally submitted. |
| add_supplementary | `path`: Complete path to the file <br><br> `name`: Display name of the file <br><br> `description`: Descriptive text about the file <br><br> `classification`: Optional classification of the file | Attach a supplementary file generated by the service to the result. The supplementary file is uploaded for the users informational use only and is not scanned. |
| drop | none | When called, the task will be dropped and will not be further processed by other remaining service(s). |
| get_param | `name`: name of the submission parameter to retrieve | Retrieve a submission parameter for the task. |
| set_service_context | `context`: Service context as string | Set the context of the service which ran the file. For example, if the service ran an AntiVirus engine on the file, then the AntiVirus definition version would be the service context. |

#### *stop* function
The `stop` function is called when the AssemblyLine service is stopped and should be used to cleanup your service. This function is optional to implement.

### Service manifest YAML
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

docker_config:
  image: cccs/assemblyline-service-tutorialservice:latest
  cpu_cores: 0.1
  ram_mb: 256

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

### Dockerfile
Create a Dockerfile file named `Dockerfile` with the following contents.
```dockerfile
FROM cccs/assemblyline-v4-service-base:latest

ENV SERVICE_PATH tutorial_service.TutorialService

# Install any service dependencies here
# For example: RUN apt-get update && apt-get install -y libyaml-dev
#              RUN pip install utils

# Switch to assemblyline user
USER assemblyline

# Copy TutorialSample service code
WORKDIR /opt/al_service
COPY . .
```

The Dockerfile is required to build a Docker container. When developing a Docker container for an AssemblyLine service,
the following must be ensured:
- The parent image must be `cccs/assemblyline-v4-service-base:latest`.
- An environment variable named `SERVICE_PATH` must be set whose value defines the Python module path to the main service
class which inherits from the `ServiceBase` class.
- Any dependency installation must be completed as the `root` user, which is set by default in the parent image. Once all 
dependency installations have been completed, you must should change the user to `assemblyline`.
- The service code and any dependency files must be copied to the `/opt/al_service` directory.

## Conclusion
After you've completed creating your first service, your `tutorial_service` directory should have the following files at minimum:

```text
.
├── Dockerfile
├── tutorial_service.py
└── service_manifest.yml
```

You are now ready to [run your service](run_your_service.md). 