# Developing an Assemblyline service

This guide has been created for developers who are looking to develop services for Assemblyline. It is aimed at individuals with general software development knowledge and basic Python skills. In-depth knowledge of the Assemblyline framework is not required to develop a service.

## Pre-requisites
Before getting started, ensure you have read through the [setup environment](../../env/getting_started/) documentation and created the appropriate development environment to perform service development.

## Build your first service
This section will guide you through the bare minimum steps required to create a running, but functionally useless service. Each sub-section below outlines the steps required for each of the different files required to create an Assemblyline service. All files created in the following sub-sections must be placed in a common directory.

!!! important
    For this documentation, we will assume that your new service directory is located at `~/git/services/assemblyline-service-sample`

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

        def load_rules(self) -> None:
            self.log.info(f'Sample will load the following rule files: {self.rules_list}')

            # Pass rules_list to module that will be consulted during analysis on execute()

        def execute(self, request):
            # ==================================================================
            # Execute a request:
            #   Every time your service receives a new file to scan, the execute function is called
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

### Service Updater Python code

In your service directory, you will first start by creating your service's updater python file. Let's use `update_server.py`.

Put the following code in your service's file:
???+ example "~/git/services/assemblyline-service-sample/update_server.py"
    ```python
    from assemblyline_v4_service.updater.updater import ServiceUpdater
    from assemblyline.odm.models.signature import Signature


    class SampleUpdateServer(ServiceUpdater):
        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)

        def import_update(self, files_sha256, client, source_name, default_classification=None):
            signatures = []
            for file, _ in files_sha256:
                # Iterate over all the files retrieved by source and upload them as signatures in Assemblyline
                with open(file, 'r') as fh:
                 signatures.append(Signature(dict(
                     classification = default_classification,
                     data = fh.read(),
                     name = f'sample_signature{len(signatures)}',
                     source = source_name,
                     status = 'DEPLOYED',
                     type = 'sample'
                 )))
            client.signatures.add_update_many(signatures)
            self.log.info(f'Successfully imported {len(signatures)} signatures')
            return

        # Service base handles the default means of gathering source updates, it's up to the service writer to define
        # how to handle importing the update into Assemblyline

        # do_source_update() of ServiceUpdater class can be overridden if necessary
        # ie. assemblyline-service-safelist


    if __name__ == '__main__':
        with SampleUpdateServer() as server:
            server.serve_forever()



### Service manifest YAML

Now that you have a better understanding of the python portion of a service. We will create the associated manifest file that holds the different configurations for the service.

In your service directory, you will add the YAML configuration file `service_manifest.yml` with the following content:
???+ example "~/git/services/assembyline-service-sample/service_manifest.yml"
    ```yaml
    # Name of the service
    name: Sample
    # Version of the service
    version: 4.0.0.dev0
    # Description of the service
    description: ALv4 Sample service from the documentation

    # Regex defining the types of files the service accepts and rejects
    accepts: .*
    rejects: empty

    # At which stage the service should run (one of FILTER, EXTRACT, CORE, SECONDARY, POST)
    # NOTE: Stages are executed in the order defined in the list
    stage: CORE
    # Which category the service is part of (one of Antivirus, Dynamic Analysis, External, Extraction, Filtering, Networking, Static Analysis)
    category: Static Analysis

    # Does the service require access to the file to perform its task
    # If set to false, the service will only have access to the file metadata (e.g. Hashes, size, type, ...)
    file_required: true
    # Maximum execution time the service has before it's considered to be timed out
    timeout: 10

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

    # Dependencies configuration block which defines:
    #   - the name of your dependencies (ie. 'updates') and its Docker configuration
    #   - Should this dependency be able to interact with the core?
    dependencies:
        updates:
            container:
                allow_internet_access: true
                command: ["python", "-m", "update_server"]
                image: ${REGISTRY}cccs/assemblyline-service-sample:$SERVICE_TAG
                ports: ["5003"]
            run_as_core: True

    # Update configuration block
    update_config:
    # list of source object from where to fetch files for update and what will be the name of those files on disk
    sources:
        - uri: https://file-examples-com.github.io/uploads/2017/02/file_example_JSON_1kb.json
        name: sample_1kb_file
    # interval in seconds at which the updater dependency runs
    update_interval_seconds: 300
    # Should the downloaded files be used to create signatures in the system
    generates_signatures: true
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
