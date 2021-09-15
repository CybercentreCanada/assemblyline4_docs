# Developing an Assemblyline service

This guide has been created for developers who are looking to develop services for Assemblyline. It is aimed at individuals with general software development knowledge and basic Python skills. In-depth knowledge of the Assemblyline framework is not required to develop a service.

## Pre-requisites
Before getting started, ensure you have read throught the [setup environment](../../env/getting_started/) documentation and created the appropriate development environment to perform service development.

## Build your first service
This section will guide you through the bare minimum steps required to create a running, but functionally useless service. Each sub-section below outlines the steps required for each of the different files required to create an Assemblyline service. All files created in the following sub-sections must be placed in a common directory.

!!! important
    For the purpose of this documentation, we will assume that your new service directory is located at: `~/git/services/assemblyline-service-sample`

### Service Python code

In your service directory, you will first start by creating your service's python file. Let's use `sample.py`.

Put the following code in your service's file:
???+ example "~/git/services/assemblyline-service-sample/sample.py"
    ```python
    from assemblyline_v4_service.common.base import ServiceBase
    from assemblyline_v4_service.common.result import Result, ResultSection

    class Sample(ServiceBase):
        def __init__(self, config=None):
            super(Sample, self).__init__(config)

        def start(self):
            # ==================================================================
            # On Startup actions:
            #   Your service might have to do some warming up on startup to make things faster

            self.log.info(f"start() from {self.service_attributes.name} service called")

        def execute(self, request):
            # ==================================================================
            # Execute a request:
            #   Every time your service receives a new file to scan, the execute function is called
            #   This is where you should execute your processing code.
            #   For the purpose of this example, we will only generate results ...
            # ==================================================================

            # 1. Create a result object where all the result sections will be saved to
            result = Result()

            # 2. Create a section to be displayed for this result
            text_section = ResultSection('Example of a default section')

            # 2.1. Add lines to your section
            text_section.add_line("This is a line displayed in the body of the section")

            # 2.2. Your section can generate a score. To do this it need to fire a heuristic.
            #     We will fire heuristic #1
            text_section.set_heuristic(1)

            # 2.3. Your section can add tags, we will add a fake one
            text_section.add_tag("network.static.domain", "cyber.gc.ca")

            # 3. Make sure you add your section to the result
            result.add_section(text_section)

            # 4. Wrap-up: Save your result object back into the request
            request.result = result
    ```

#### *ServiceBase* class
As you can see in the previous example, the main class of your Assemblyline service must inherit from the `ServiceBase` class which can be imported from `assemblyline_v4_service.common.base`. The `ServiceBase` base class includes many instance variables which can be used to access service related information.

##### Class variables
The following tables describes all of the available variables of the `ServiceBase` class.

| Variable Name | Description |
|:---|:---|
| config | Reference to the service parameters containing values updated by the user for service configuration. |
| log | Reference to the logger. |
| service_attributes | Service attributes from the [service manifest](service_manifest.md). |
| working_directory | Returns the directory path which the service can use to temporarily store files during each task execution. |

##### get_api_interface()
The purpose of the `get_api_interface` function is to give the service direct access to the `Service Server` component to perform API request to find out if a file or a tag is meant to be safelisted.

!!! warning
    By using this function, your service will not work using the `run_service_once` command and will be completely tied to the `service server` API.

##### get_tool_version()
The purpose of the `get_tool_version` function is to the return a string of all the tools used by the service. The tool version should be updated to reflect changes in the service tools, so that Assemblyline can rescan files on the new service version if they are submitted again.

##### start()
The `start` function is called when the Assemblyline service is initiated and should be used to prepare your service for task execution. This function is optional to implement.

##### execute()
The `execute` function is called every time the service receives a new file to scan. This is where you should execute your processing code. In the example above, we only generate static results for illustration purposes.

The `request` object is provided as an input to the `execute` function. The `request` object holds information about the task to be processed by the service. The following table describes all the `request` object variables which the service can use.

| Variable name | Description |
|:---|:---|
| deep_scan | Returns whether the file should be deep scanned or not. Deep scanning usually takes more time and is better suited for file sent manually. |
| file_contents | Returns the raw bytes content of the file to be scanned. |
| file_name | Returns the name of the file (as submitted by the user) to be scanned. |
| file_path | Returns the path to the file to be scanned. The service can use this path directly to access the file. |
| file_type | Returns the Assemblyline style file type of the file to be scanned. |
| max_extracted | Returns the max number of files that are allowed to be extracted by a service. By default this is set to 500. |
| md5 | Returns the MD5 of the file to be scanned. |
| result | Used to set and get the current result. |
| sha1 | Returns the SHA1 of the file to be scanned. |
| sha256 | Returns the SHA256 of the file to be scanned. |
| sid | ID of the submission being scanned. |
| task | The original task object used to create this request. You can find more information there about the request (metadata submitted, files already extracted by other services, tags already generated by other services and more...) |
| temp_submission_data | Can be used to get and set temporary submission data which is passed onto subsequent tasks resulting from adding extracted files. |

The following table describes all the `request` object functions which the service can use.

| Function name | Parameters | Description |
|:---|:---|:---|
| add_extracted | `path`: Complete path to the file <br><br> `name`: Display name of the file <br><br> `description`: Descriptive text about the file <br><br> `classification`: Optional classification of the file | Attach a file extracted by the service to the result. The extracted file will also be scanned through a set of services, as if it had been originally submitted. |
| add_supplementary | `path`: Complete path to the file <br><br> `name`: Display name of the file <br><br> `description`: Descriptive text about the file <br><br> `classification`: Optional classification of the file | Attach a supplementary file generated by the service to the result. The supplementary file is uploaded for the users informational use only and is not scanned. |
| drop | none | When called, the task will be dropped and will not be further processed by other remaining service(s). |
| get_param | `name`: name of the submission parameter to retrieve | Retrieve a submission parameter for the task. |
| set_service_context | `context`: Service context as string | Set the context of the service which ran the file. For example, if the service ran an AntiVirus engine on the file, then the AntiVirus definition version would be the service context. |

##### stop()
The `stop` function is called when the Assemblyline service is stopped and should be used to cleanup your service. This function is optional to implement.

### Service manifest YAML

Now that you have oa better understanding of the python portion of a service. We will create the associated manifest file that hold the different configuration for the service.

In your service directory, you will add the YAML configuration file `service_manifest.yml` with the following content:
???+ example "~/git/services/assembyline-service-sample/service_manifest.yml"
    ```yaml
    # Name of the service
    name: Sample
    # Version of the service
    version: 1
    # Description of the service
    description: ALv4 Sample service from the documentation

    # Regex defining the types of files the service accepts and rejects
    accepts: .*
    rejects: empty

    # At which stage the service should run (one of: FILTER, EXTRACT, CORE, SECONDARY, POST)
    # NOTE: Stages are executed in the order defined in the list
    stage: CORE
    # Which category the service is part of (one of: Antivirus, Dynamic Analysis, External, Extraction, Filtering, Networking, Static Analysis)
    category: Static Analysis

    # Does the service require access to the file to perform its task
    # If set to false, the service will only have access to the file metadata (e.g. Hashes, size, type, ...)
    file_required: true
    # Maximum execution time the service has before it's considered to be timed out
    timeout: 10

    # is the service enabled by default
    enabled: true

    # Service heuristic blocks: List of heuristics object that define the different heuristics used in the service
    heuristics:
      - description: This is a demo heuristic
        filetype: "*"
        heur_id: 1
        name: Demo
        score: 100

    # Docker configuration block which defines:
    #  - the name of the docker container that will be created
    #  - cpu and ram allocation by the container
    docker_config:
      image: ${REGISTRY}testing/assemblyline-service-sample:latest
      cpu_cores: 1.0
      ram_mb: 1024
    ```

!!! important
    The `service_manifest.yml` has a lot more configurable parameters that you might be required to change depending of the service you are building. You should get familiar with the complete list by reading the [service manifest](../service_manifest) documentation.

### Dockerfile
Finally, the last file needed to complete your assemblyline service is the `Dockerfile` that is used to create it's docker container.

In your service directory, create a file named `Dockerfile` with the following content:
???+ example "~/git/services/assemblyline-service-sample/Dockerfile"
    ```dockerfile
    FROM cccs/assemblyline-v4-service-base:stable

    # Python path to the service class from your service directory
    #  The following example refers to the class "Sample" from the "sample.py" file
    ENV SERVICE_PATH sample.Sample

    # Install any service dependencies here
    # For example: RUN apt-get update && apt-get install -y libyaml-dev
    #              RUN pip install utils

    # Switch to assemblyline user
    USER assemblyline

    # Copy Sample service code
    WORKDIR /opt/al_service
    COPY . .
    ```

The Dockerfile is required to build a Docker container. When developing a Docker container for an Assemblyline service,
the following must be ensured:

- The parent image must be `cccs/assemblyline-v4-service-base:stable` so your are using a stable build of service base.
- An environment variable named `SERVICE_PATH` must be set whose value defines the Python module path to the main service
class which inherits from the `ServiceBase` class.
- Any dependency installation must be completed as the `root` user, which is set by default in the parent image. Once all
dependency installations have been completed, you must should change the user to `assemblyline`.
- The service code and any dependency files must be copied to the `/opt/al_service` directory.

## Conclusion
After you've completed creating your first service, your `~/git/services/assemblyline-service-sample` directory should have the following files at minimum:

```text
.
├── Dockerfile
├── result_sample.py
└── service_manifest.yml
```
