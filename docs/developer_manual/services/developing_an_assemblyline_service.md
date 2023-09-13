# Developing an Assemblyline service

This guide has been created for developers who are looking to develop services for Assemblyline. It is aimed at individuals with general software development knowledge and basic Python skills. In-depth knowledge of the Assemblyline framework is not required to develop a service. You should understand the concepts found [here](../../user_manual/results) though.

## Pre-requisites
Before getting started, ensure you have read through the [setup environment](../../env/getting_started/) documentation and created the appropriate development environment to perform service development.

## Build your first service
This section will guide you through the bare minimum steps required to create a running, but functionally useless service. Each sub-section below outlines the steps required for each of the different files required to create an Assemblyline service. All files created in the following sub-sections must be placed in a common directory.

!!! important
    For this documentation, we will assume that your new service directory is located at `~/git/services/assemblyline-service-sample`

To build a service, we need a minimum of three files. 

1. A Python script that has a class implementing `ServiceBase` and the `execute` function
2. A service manifest
3. A Dockerfile. 

The Dockerfile is not needed for development but will be required to deploy to your service to an Assemblyline instance.

### Service Python code

In your service directory, you will first start by creating your service's python file. Let's use `result_sample.py`.

Put the following code in your service's file:
???+ example "~/git/services/assemblyline-service-sample/result_sample.py"
    ```python
    from assemblyline_v4_service.common.base import ServiceBase
    from assemblyline_v4_service.common.request import ServiceRequest
    from assemblyline_v4_service.common.result import Result, ResultSection

    class Sample(ServiceBase):
        def __init__(self, config=None):
            super(Sample, self).__init__(config)

        def start(self):
            # ==================================================================
            # Startup actions:
            #   Your service might have to do some warming up on startup to make things faster
            # ==================================================================

            self.log.info(f"start() from {self.service_attributes.name} service called")

        def execute(self, request: ServiceRequest) -> None:
            # ==================================================================
            # Execute a request:
            #   Every time your service receives a new file to scan, the execute function is called.
            #   This is where you should execute your processing code.
            #   For this example, we will only generate results ...
            # ==================================================================

            # 1. Create a result object where all the result sections will be saved to
            result = Result()

            # 2. Create a section to be displayed for this result
            text_section = ResultSection('Example of a default section')

            # 2.1. Add lines to your section
            text_section.add_line("This is a line displayed in the body of the section")

            # 2.2. Your section can generate a score. To do this it needs to fire a heuristic.
            #     We will fire heuristic #1
            text_section.set_heuristic(1)

            # 2.3. Your section can add tags, we will add a fake one
            text_section.add_tag("network.static.domain", "cyber.gc.ca")

            # 3. Make sure you add your section to the result
            result.add_section(text_section)

            # 4. Wrap-up: Save your result object back into the request
            request.result = result
    ```

### Service manifest YAML

Now that you have a better understanding of the python portion of a service. We will create the associated manifest file that holds the different configurations for the service.

In your service directory, you will add the YAML configuration file `service_manifest.yml` with the following content:
???+ example "~/git/services/assembyline-service-sample/service_manifest.yml"
    ```yaml
    # Name of the service
    name: Sample
    # Version of the service
    version: 4.4.0.dev0
    # Description of the service
    description: ALv4 Sample service from the documentation

    # Regex defining the types of files the service accepts and rejects
    accepts: .*
    rejects: empty

    # At which stage the service should run (one of FILTER, EXTRACT, CORE, SECONDARY, POST, REVIEW)
    # NOTE: Stages are executed in the order defined in the list
    stage: CORE
    # Which category the service is part of (one of Antivirus, Dynamic Analysis, External, Extraction, Filtering, Internet Connected, Networking, Static Analysis)
    category: Static Analysis

    # Does the service require access to the file to perform its task
    # If set to false, the service will only have access to the file metadata (e.g. Hashes, size, type, ...)
    file_required: true
    # Maximum execution time the service has before it's considered to be timed out
    timeout: 60

    # is the service enabled by default
    enabled: true

    # Service heuristic blocks: List of heuristic objects that define the different heuristics used in the service
    heuristics:
      - description: This is a demo heuristic
        filetype: "*"
        heur_id: 1
        name: Demo
        score: 100

    # Docker configuration block which defines:
    #  - the name of the docker container that will be created
    #  - CPU and ram allocation by the container
    docker_config:
      image: ${REGISTRY}testing/assemblyline-service-sample:latest
      cpu_cores: 1.0
      ram_mb: 1024
    ```

!!! important
    The `service_manifest.yml` has a lot more configurable parameters that you might be required to change depending on the service you are building. You should get familiar with the complete list by reading the [service manifest](../advanced/service_manifest) advanced documentation.

### Dockerfile
Finally, the last file needed to complete your assemblyline service is the `Dockerfile` that is used to create its Docker container.

In your service directory, create a file named `Dockerfile` with the following content:
???+ example "~/git/services/assemblyline-service-sample/Dockerfile"
    ```dockerfile
    FROM cccs/assemblyline-v4-service-base:stable

    # Python path to the service class from your service directory
    #  The following example refers to the class "Sample" from the "sample.py" file
    ENV SERVICE_PATH result_sample.Sample

    # Install any service dependencies here
    # For example: RUN apt-get update && apt-get install -y libyaml-dev
    #
    #              COPY requirements.txt requirements.txt
    #              RUN pip install --no-cache-dir --user --requirement requirements.txt && rm -rf ~/.cache/pip

    # Switch to assemblyline user
    USER assemblyline

    # Copy Sample service code
    WORKDIR /opt/al_service
    COPY . .
    ```

The Dockerfile is required to build a Docker container. When developing a Docker container for an Assemblyline service,
the following must be ensured:

- The parent image must be `cccs/assemblyline-v4-service-base:stable` so you are using a stable build of service base.
- An environment variable named `SERVICE_PATH` must be set whose value defines the Python module path to the main service
class which inherits from the `ServiceBase` class.
- Any dependency installation must be completed as the `root` user, which is set by default in the parent image. Once all
dependency installations have been completed, you must change the user to `assemblyline`.
- The service code and any dependency files must be copied to the `/opt/al_service` directory.

## Conclusion
After you've completed creating your first service, your `~/git/services/assemblyline-service-sample` directory should have the following files at a minimum:

```text
.
├── Dockerfile
├── result_sample.py
└── service_manifest.yml
```

### Good service examples

#### [ElfParser](https://github.com/CybercentreCanada/assemblyline-service-elfparser)
- Package a compiled executable from https://github.com/jacob-baines/elfparser
- Parse the output of the executable to fill `ResultSections` for the user

#### [ApiVector](https://github.com/CybercentreCanada/assemblyline-service-apivector)
- Use a public library ([`apiscout`](https://github.com/danielplohmann/apiscout), [`lief`](https://github.com/lief-project/LIEF))
- Load an external file
- Use an updater

#### [UrlDownloader](https://github.com/CybercentreCanada/assemblyline-service-urldownloader/)
Unique key-value usage in the service manifest relative to the average service:
- `stage: POST`
- `file_required: false`
- `is_external, allow_internet_access: true`
- `uses_tag_scores, uses_metadata, uses_temp_submission_data: true`
