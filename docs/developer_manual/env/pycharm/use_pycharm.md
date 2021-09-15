# Use PyCharm

Here are some pointers on how to run most of the core components live in PyCharm when either your local or remote development VM is ready.

## Load docker-compose files

### Minimal Dependencies

When dependencies are loaded, you can launch any core components and its dependencies should be satisfied.

=== "Pycharm Professional"

    1. In your project file viewer
    2. Browse to ```assemblyline-base/dev/depends```
    3. Right click on ```docker-compose-minimal.yml```
    4. Select ```Create 'depends/docker-compose...'```
    5. Click ```OK```
    6. You'll notice on the top right that a new Compose deployment has been created for dependencies, hit the play button next to it to launch the containers.

    !!! tip
        From now on you can just select that Compose deployment for dependencies from the dropdown up top and hit the run button to launch it. It is good practice to Edit the configurations and give it a proper name.

=== "Docker-compose (Pycharm Community)"

    In a new terminal on the VM, run the following commands:

    ```shell
    cd ~/git/alv4/assemblyline-base/dev/depends/
    sudo docker-compose -f docker-compose-minimal.yml up
    ```

    !!! tip
        This will take over the terminal and the logs will be displayed there. Hit ```ctrl-c``` when you want to shut the containers down.

    If you close the terminal and the containers keep running, run the following commands to shut down the containers:

    ```shell
    cd ~/git/alv4/assemblyline-base/dev/depends/
    sudo docker-compose -f docker-compose-minimal.yml down
    ```

### Core services

Core services depend on the dependencies in the docker-compose file. When the core services are loaded you should be able to point a browser at ```https://IP_OF_VM``` and you'll be greeted with a working version of Assemblyline with no services configured.

!!! note
    Default admin user credentials are:

    * Username: ```admin```
    * Password: ```admin```

=== "Pycharm Professional"

    1. In your project file viewer
    2. Browse to `assemblyline-base/dev/core`
    3. Right click on `docker-compose.yml`
    4. Select `Create 'core: Compose Deploy...'`
    5. Click `OK`
    6. You'll notice on the top right that a new Compose deployment has been created for core services, hit the play button next to it to launch the containers.

    !!! tip
        From now on you can just select that Compose deployment for core components from the dropdown up top and hit the run button to launch it. It is good practice to Edit the configurations and give it a proper name.

=== "Docker-compose (Pycharm Community)"

    In a new terminal on the VM, run the following commands:
    ```shell
    cd ~/git/alv4/assemblyline-base/dev/core/
    sudo docker-compose up
    ```

    !!! tip
        This will take over the terminal and the logs will be displayed there. Hit ```ctrl-c``` when you want to shut the containers down.

    If you close the terminal and the containers keep running, run the following commands to shut down the containers:
    ```shell
    cd ~/git/alv4/assemblyline-base/dev/core/
    sudo docker-compose down
    ```

## Load single container live

TBD

## Run components live from PyCharm

### Core Components

Most of the core components are going to be as easy to run as finding the component main file then hit `Run` or `Debug`... All core components require you to run the ```Minimal Dependencies``` docker-compose file prior to launch them.

!!! example "Launching the API Server"
    1. Find the UI main file in the project files browser
        - ```assemblyline-ui/assemblyline_ui/app.py```
    2. Right click on it
    3. Select either ```Run 'app'``` or ```Debug 'app'```

### Live Services

The main exception to very easy-to-run components are debugging services live in the system. Service require you to run two separate components to process files. The two components talk via named pipes in the /tmp folder. Inside the Docker container this is very easy but debugging this live needs requires some sacrifices.

!!! warning "Limitations"
    1. You can only run one live debugging service at the time
    2. Depending on where you stop them and how you stop them, they don't fully cleanup after themselves.
    3. They require extra configuration, it's not just click and go

We will show you how to run the Sample service live in the system created in the [developing and Assemblyline service](../../../services/developing_an_assemblyline_service) documentation. This requires you to run the full infrastructure using the docker-compose files prior to execute the steps. ([Minimal dependencies](#minimal-dependencies) and [Core services](#core-services))

!!! example
    Setup `Task handler` run configuration if does not exist:

    1. Inside the Pycharm window pointing to the services (`~/git/services`), you should see an `Add configuration` button, click on it.
    2. Click the `+` button at the top left
    3. Click `Python`
    4. First option in the configuration tab is a drop down that says `Script path:` click it and choose `Module Name:`
    5. In the box beside it, write: `assemblyline_service_client.task_handler`
    6. In the name box at the top, write `Task handler`
    7. Click the `OK` Button

    Now if you click the green play button beside the newly created `Task handler` run configuration, Task handler will be waiting for the service to start.

    Setup the new service configuration:

    1. Click on the dropdown menu that has your newly created run configuration (`Task handler`) then click `Edit configurations...`
    2. Click the `+` button at the top left
    3. Click `Python`
    4. First option in the configuration tab is a drop down that says `Script path:` click it and choose `Module Name:`
    5. In the box beside it, write: `assemblyline_v4_service.run_service`
    6. Add `;SERVICE_PATH=sample.Sample` to the `Environment variables`
    7. Set the working directory to: `assemblyline-service-sample` by pressing the little `folder` on the right and browsing to that directory
    8. In the name box at the top, write `Sample Service - LIVE`
    9. Click `OK`
    10. Now that dropdown near the top says `Sample Service - LIVE`, click the `play` or the `bug` button to either `Run` or `Debug` the service
        - In the Run or Debug window, the service will be stuck at *Waiting for receive task named pipe to be ready...*. This is because task_handler shutdown after registering the service for the first time.
    11. Select `task_handler` from the dropdown near the top and click the `play` or `bug` button beside it again.
        - Now both `Sample Service - LIVE` and `Task handler` will stay up an poll for task from service server.
