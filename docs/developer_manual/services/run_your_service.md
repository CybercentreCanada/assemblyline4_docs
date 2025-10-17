# Run your service

This section of the service documentation will show you how to run your service in 3 different ways:

1. Standalone mode directly on a file
2. Live code on an Assemblyline system
3. Production container on an Assemblyline system

!!! important
    This documentation assumes the following:

    1. You have read through the [setup environment](../../env/getting_started/) documentation and created the appropriate development environment to perform service development.
    2. You have completed the [developing an Assemblyline service](../developing_an_assemblyline_service) documentation and your service is in the `~/git/services/assemblyline-service-sample` directory on the machine where your IDE runs.
    3. Whether you are using VSCode or PyCharm as your IDE, you have a virtual environment dedicated to running services either located at `~/git/services/venv` or `~/venv/services` on the VM where the code runs

## Standalone mode

To test an Assemblyline service in standalone mode, the `run_service_once` module from the [assemblyline-v4-service](https://pypi.org/project/assemblyline-v4-service/) package can be used to run a single task through the service for testing.

### Running the Sample service

Load the virtual environment

```shell
source ~/git/services/venv/bin/activate
```

Ensure the current working directory is the root of the service directory of the service to be run.

```shell
cd ~/git/services/assemblyline-service-sample
```

From a terminal, run the `run_service_once` module, specifying the service path for the service to run and the path to the file to scan. For this example, we will have the service scan itself.

```shell
python -m assemblyline_v4_service.dev.run_service_once result_sample.Sample result_sample.py
```

The `run_service_once` module creates a directory at the same spot where the file is found with the service name that scanned the file appended to it. In the previous example, the output of the service should be located at `~/git/services/assemblyline-service-sample/sample.py_sample`. The directory will contain a `result.json` file containing the result from the service.

You can view the `result.json` file using the following command:

```shell
cat ~/git/services/assemblyline-service-sample/result_sample.py_sample/result.json | json_pp
```

It will look something like this:
!!! example "~/git/services/assemblyline-service-sample/sample.py_sample/result.json pretty printed"
    ```json
    {
        "classification" : "TLP:W",
        "drop_file" : false,
        "response" : {
            "extracted" : [],
            "milestones" : {
                "service_completed" : "2021-09-15T15:52:47.024909Z",
                "service_started" : "2021-09-15T15:52:47.024761Z"
            },
            "service_context" : null,
            "service_debug_info" : null,
            "service_name" : "sample",
            "service_tool_version" : null,
            "service_version" : "1",
            "supplementary" : []
        },
        "result" : {
            "score" : 100,
            "sections" : [
                {
                    "auto_collapse" : false,
                    "body" : "This is a line displayed in the body of the section",
                    "body_format" : "TEXT",
                    "classification" : "TLP:W",
                    "depth" : 0,
                    "heuristic" : {
                    "attack_ids" : [],
                    "frequency" : 1,
                    "heur_id" : 1,
                    "score" : 100,
                    "score_map" : {},
                    "signatures" : {}
                    },
                    "tags" : {
                    "network" : {
                        "static" : {
                            "domain" : [
                                "cyber.gc.ca"
                            ]
                        }
                    }
                    },
                    "title_text" : "Example of a default section",
                    "zeroize_on_tag_safe" : false
                }
            ]
        },
        "sha256" : "dab595d88c22eba68e831e072471b02206d12cb29173c708c606416ecf50b942",
        "temp_submission_data" : {}
    }
    ```

Using the standalone mode via `run_service_once` is going to help quickly catch any bug, but won't allow you to see the UI elements.

## Live mode

The following technique is how to hook in a service to an Assemblyline development instance so you can perform live debugging and you can send files to your service using the Assemblyline UI.

The way to run a service in debug mode will differ depending on if you've been using VSCode or PyCharm as your IDE. Follow the appropriate documentation for your current setup:

* [Run a service LIVE in VSCode](../../env/vscode/use_vscode/#running-live-services)
* [Run a service LIVE in PyCharm](../../env/pycharm/use_pycharm/#live-services)

!!! important
    You will need to adjust the documentation according to:

    1. The correct name for your service (`Sample`)
    2. The correct service python module for your service (`sample.Sample`)
    3. The correct working directory for your service (`~/git/services/assemblyline-service-sample`)

### Run from a shell

If you don't plan on doing any debugging and you just want to run the service live in your development environment, you can just spin up two shells and run `Task Handler` in one and your `Sample service` in the other.

#### Task Handler

```shell
# Load your service virtual environment
source ~/git/services/venv/bin/activate

# Run task handler
python -m assemblyline_service_client.task_handler
```

#### Sample service

```shell
# Load your service virtual environment
source ~/git/services/venv/bin/activate

# Go to your service directory
cd ~/git/services/assemblyline-service-sample

# Run your service
SERVICE_PATH=result_sample.Sample python -m assemblyline_v4_service.run_service
```

## Production container mode

When you are confident your service is stable enough, it is time to test it in its final form: A Docker container.

### Build the container

Change working directory to root of the service:

```bash
cd ~/git/services/assemblyline-service-sample
```

Run the `docker build` command and tag the container with the same name that the container name has in your service manifest

```bash
docker build -t testing/assemblyline-service-sample .
```

### Run the container LIVE

The way to run a container LIVE in your development environment differs depending on if you've been using VSCode or PyCharm as your IDE. Follow the appropriate documentation for your current setup:

* [Run a single container in VSCode](../../env/vscode/use_vscode/#resultsample)
* [Run a single container in PyCharm](../../env/pycharm/use_pycharm/#load-single-container-live)

!!! important
    You will need to adjust the documentation according to:

    1. The correct name for your service (`Sample`)
    2. The correct container name (`testing/assemblyline-service-sample`)

#### Run container from a shell

If you don't want to use the IDE to test your production container, you can always run it straight from a shell.

Use the following command to run it:

```shell
docker run --env SERVICE_API_HOST=http://`ip addr show docker0 | grep "inet " | awk '{print $2}' | cut -f1 -d"/"`:5003 --network=host --name SampleService testing/assemblyline-service-sample
```

### Add the container to your deployment

!!! note
    For the scaler and updater to be able to use your service container, they must be able to get it from a docker registry. You can either use DockerHub, a work central registry, or a local registry.

#### Push the container to your local registry

!!! tip
    A local docker registry should have already been installed during the [development environment setup](../../env/getting_started). If not, you can start one by simply running this command:

    ```shell
    sudo docker run -dp 32000:5000 --restart=always --name registry registry
    ```

Use the following to push your sample service to the local registry:

```shell
docker tag testing/assemblyline-service-sample localhost:32000/testing/assemblyline-service-sample
docker push --all-tags localhost:32000/testing/assemblyline-service-sample
```

#### Add the service in the management interface

1.  Using your web browser, go to the service management page: [https://localhost/admin/services](https://localhost/admin/services) (Replace localhost by your VM's IP)
2.  Click the `Add service` button
3.  Paste the entire content of the `service_manifest.yml` file from your service directory in the text box
    * If you are using the local registry from this documentation, change the `${REGISTRY}` from the content of `service_manifest.yml` to `host.docker.internal:32000/`
4.  Click the `Add` button

Your service information has been added to the system. The scaler component should automatically start a container of your newly created service.
