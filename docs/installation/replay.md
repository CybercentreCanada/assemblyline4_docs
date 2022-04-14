# Assemblyline Replay

Replay is a new feature of Assemblyline that allows you to bridge the gap between online and offline deployments of Assemblyline. You will be able to start a scan on your online deployment, bundle the results, ship them to your offline enclave and continue the scans in the offline enclave.

## How it works

Replay has two main component:

* Replay Creator
* Replay Loader

The `Replay Creator` component will monitor either the alerts, the submissions or both and bundle them once they are completed including all their results, errors, files, etc... Those bundles will have to be transfered with the method of your choice to your offline enclave where the `Replay Loader` component will monitor a directory loading the bundles into your offline instance. Once these bundles are transfered onto your target system, the loader will be able to tell your offline instance to resume the scan and rescan the files with your offline services if you tell it to.

![Replay diagram](./images/Replay.png)

## Replay configuration file

## Helm Deployment

There are multiple ways to configure replay so unfortunately there is no one size fits all configuration. The easiest setup by far is to use the different flags that we have pre-configured in our helm chart. Make sure you pull our latest helm chart to be able to benefit of this easy setup.

The helm chart deployment of replay will automatically scale the worker components if there are too many bundles to create or load.

### Configure the source instance

The source instance is the one that will run the Replay Creator components. These components will have to write the alert and submission bundle somewhere. Convinently, replay uses the filestore feature of Assemblyline so you can use any type of storage that it supports to save your files.

* FTP
* S3
* Azure blob
* ...

When you know where your files are going to be saved to, edit the `values.yml` file of your helm deployment and add the following lines:

!!! example "Partial values.yaml config for replay creator setup"
    ```yaml
    ...

    # Turn on Replay
    useReplay: true

    # Set replay mode to creator
    replayMode: "creator"

    # Set the output folder for the creator worker
    replay:
      creator:
        output_filestore: <AN ASSEMBLYLINE FILESTORE LINK - SEE REPLAY CONFIGURATION FILE>

    ...
    ```

When this is done you can just perform an Helm upgrade to apply the changes to your instance.

### Configure the target instance

The target instance is the one that will run the Replay Loader components. These components will have to have direct access to a directory where the files transfered over are dropped. This limits what can be use so `NFS` is our recommended way for now.

Edit the `values.yml` file of your helm deployment and add the following lines:
!!! example "Partial values.yaml config for replay loader setup"
    ```yaml
    ...

    # Turn on Replay
    useReplay: true

    # Set replay mode to loader
    replayMode: "loader"

    # Make sure input folder is accessible
    replayLivenessCommand: "ls /tmp/replay/input"

    # Load input folder
    replayLoaderVolume:
      - name: replay-data
        nfs:
        server: localhost
        path: /path/to/folder

    ...
    ```

When this is done you can just perform an Helm upgrade to apply the changes to your instance.

## Alternate Deployment

If you are not using our helm chart to deploy Assemblyline, perhaps you are using a docker-compose deployment, there are alternatives on how to run the different replay components

=== "Virtual Env using API"

    If you are using the API mode to launch replay, you do not have to launch replay in a container and can just use a normal python process in a virtual environment if you choose so.

    !!! important

        * You will need python 3.9+ to run the Assemblyline code, if you don't have it, use the container mode.
        * The user using the replay creator needs an API Key with read access where as the user using the replay loader needs an API Key with write access.
        * Because you are launching replay directly as a process the replay.yml configuration has to be in `/etc/assemblyline/` folder.

    First of all, create your virtual environment and install the pre-requisite packages:
    ```shell
    python3.9 -m venv ~/replay_venv
    ~/replay_venv/bin/pip3.9 install assemblyline assemblyline-core assemblyline_client
    ```

    Then you can create your `/etc/assemblyline/replay.yml` configuration file and tell replay to use the API mode:

    !!! example "Partial replay.yml configuration"
        ```yaml
        creator:
          client:
            type: api
            options:
              host: "https://<YOUR SOURCE SERVER DOMAIN/IP>:443",
              apikey: "<PASTE YOUR SOURCE API KEY>"
              user: <YOUR SOURCE USERNAME>
              verify: false
        loader:
          client:
            type: api
            options:
              host: "https://<YOUR SOURCE SERVER DOMAIN/IP>:443",
              apikey: "<PASTE YOUR SOURCE API KEY>"
              user: <YOUR SOURCE USERNAME>
              verify: false
        ```

    Finally, just run which ever components are relevant depending on the Assemblyline instance you are on:

    === "Source Assemblyline instance"

        Run the creator:
        ```shell
        ~/replay_venv/bin/python3.9 -m assemblyline_core.replay.creator.run
        ```
        And its worker:
        ```shell
        ~/replay_venv/bin/python3.9 -m assemblyline_core.replay.creator.run_worker
        ```

    === "Target Assemblyline instance"

        Run the loader:
        ```shell
        ~/replay_venv/bin/python3.9 -m assemblyline_core.replay.loader.run
        ```
        And its worker:
        ```shell
        ~/replay_venv/bin/python3.9 -m assemblyline_core.replay.loader.run_worker
        ```

=== "Docker container using API"

    If you are using the API mode to launch replay, you will not be able to use the already existing Assemblyline containers to use Replay, you will have to build your own.

    !!! important

        * The user using the replay creator needs an API Key with read access where as the user using the replay loader needs an API Key with write access.
        * We will assume the configuration files for replay are in a `replay_config` folder in you home directory.

    With the following Dockerfile in `~/replay_config/Dockerfile`:

    ```Dockerfile
    FROM cccs/assemblyline-core:stable

    pip install --no-cache-dir --user assemblyline_client
    ```

    Build your replay container:

    ```shell
    (cd ~/replay_config/ && docker build --no-cache -t cccs/assemblyline-replay .)
    ```

    Then you can create your `~/replay_config/replay.yml` configuration file and tell replay to use the API mode:

    !!! example "Partial replay.yml configuration"
        ```yaml
        creator:
          client:
            type: api
            options:
              host: "https://<YOUR SOURCE SERVER DOMAIN/IP>:443",
              apikey: "<PASTE YOUR SOURCE API KEY>"
              user: <YOUR SOURCE USERNAME>
              verify: false
        loader:
          client:
            type: api
            options:
              host: "https://<YOUR SOURCE SERVER DOMAIN/IP>:443",
              apikey: "<PASTE YOUR SOURCE API KEY>"
              user: <YOUR SOURCE USERNAME>
              verify: false
        ```

    Finally, just run which ever components are relevant depending on the Assemblyline instance you are on:

    === "Source Assemblyline instance"

        Run the creator:
        ```shell
        docker run --rm \
        --name assemblyline-replay-creator \
        -v ~/replay_config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.creator.run
        ```
        And its worker:
        ```shell
        docker run --rm \
        --name assemblyline-replay-creator-worker \
        -v ~/replay_config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.creator.run_worker
        ```

    === "Target Assemblyline instance"

        Run the loader:
        ```shell
        docker run --rm \
        --name assemblyline-replay-loader \
        -v ~/replay_config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.loader.run
        ```
        And its worker:
        ```shell
        docker run --rm \
        --name assemblyline-replay-loader-worker \
        -v ~/replay_config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.loader.run_worker
        ```

=== "Docker container direct DB Access"

    This scenario is mainly going to be used if you have a docker-compose appliance of Assemblyline so you can just run a couple extra containers to get replay going. Since you have direct access to Assemblyline's Database and queues you do not have to create a special container and can just use the core container.

    !!! important
        We will assume that all you are running an appliance out of you home folder (`~/appliance/`). You should adjust the paths correctly to fit your installation.

    First of all, you will need to create your `replay.yml` file in your `~/appliance/config` directory:

    !!! example "Partial replay.yml configuration"
        ```yaml
        creator:
          client:
            type: direct
        loader:
          client:
            type: direct
        ```

    Then you can just run which ever components are relevant depending on the Assemblyline instance you are on:

    === "Source Assemblyline instance"

        Run the creator:
        ```shell
        docker run --rm \
        --name assemblyline-replay-creator \
        -v ~/appliance/config/classification.yml:/etc/assemblyline/classification.yml:ro \
        -v ~/appliance/config/config.yml:/etc/assemblyline/config.yml:ro \
        -v ~/appliance/config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.creator.run
        ```
        And its worker:
        ```shell
        docker run --rm \
        --name assemblyline-replay-creator-worker \
        -v ~/appliance/config/classification.yml:/etc/assemblyline/classification.yml:ro \
        -v ~/appliance/config/config.yml:/etc/assemblyline/config.yml:ro \
        -v ~/appliance/config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.creator.run_worker
        ```

    === "Target Assemblyline instance"

        Run the loader:
        ```shell
        docker run --rm \
        --name assemblyline-replay-loader \
        -v ~/appliance/config/classification.yml:/etc/assemblyline/classification.yml:ro \
        -v ~/appliance/config/config.yml:/etc/assemblyline/config.yml:ro \
        -v ~/appliance/config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.loader.run
        ```
        And its worker:
        ```shell
        docker run --rm \
        --name assemblyline-replay-loader-worker \
        -v ~/appliance/config/classification.yml:/etc/assemblyline/classification.yml:ro \
        -v ~/appliance/config/config.yml:/etc/assemblyline/config.yml:ro \
        -v ~/appliance/config/replay.yml:/etc/assemblyline/replay.yml:ro \
        -v /tmp/replay/files:/tmp/replay/output \
        -v /tmp/replay/creator:/tmp/replay/work \
        cccs/assemblyline-core:stable \
        python -m assemblyline_core.replay.loader.run_worker
        ```
