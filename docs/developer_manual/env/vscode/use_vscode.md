# Use VSCode

Once the setup script is run, your installation of VSCode will be entirely ready to run and debug any components of Assemblyline and most launch targets are already pre-configured. This page will point you in the right direction to perform some of the more common task that you'll have to do.

## Running Tasks

After all recommended packaged are install in VSCode, the task button on the side bar should be revealed and will give you a quick access to the most important tasks in the system.

![Task Explorer](./images/task_explorer.png)

Those task are splitted into 3 categories:

1. Container - Run single container for a single task
2. Docker compose - Execute a set of containers for a specific task dependency
3. Pytest dependencies - Run necessary dependencies to run tests

!!! tip
    You can edit the task list by modifying the ```.vscode/tasks.json```. The default `task.json` file can be found in the [assemblyline-development-setup](https://github.com/CybercentreCanada/assemblyline-development-setup/blob/master/.vscode/tasks.json) repository.

### Container tasks

Container tasks execute one specific container in the system. Two of theses tasks are predefined:

#### Frontend

The Frontend task is used to run the User Interface of assemblyline. It is only useful for when you launch the Assemblyline API Server in debug mode and are **NOT** using the ```core components``` task. Assemblyline's frontend is now built in Reactjs and is served via ```NPM serve``` which is why it is not part of this setup. Refer to the [frontend development](../../../frontend/frontend) page for more info on how to do development on Assemblyline frontend.

#### ResultSample

The ResultSample task was created to show the developers how to run newly created service containers in the system.

!!! example "Deep dive in the ResultSample Task"
    This is what the JSON block for executing the ResultSample service looks like:
    ```json
    ...
    {
        "label": "Container - ResultSample",
        "type": "shell",
        "options": {
            "env": {
                "LOCAL_IP": "172.17.0.1"
            }
        },
        "command": "docker run --env SERVICE_API_HOST=http://${LOCAL_IP}:5003 --network=host cccs/assemblyline-service-resultsample",
        "runOptions": {
            "instanceLimit": 1
        }
    },
    ...
    ```

    Essentially, all this does is it runs the ```docker run``` command and specify where the *service server* API is located. You can change the ```LOCAL_IP``` environment variable if your docker subnet is different.

### Docker compose tasks

The docker compose tasks are use to run sets of predefined dependencies in the system.

#### Dependencies

Before trying to run anything, at the bare minimum you will have to have one of the two ```Dependencies``` tasks running.

* ```Dependencies (Basic)``` will run the strict minimum containers to start components in the system: Elasticsearch, Redis, Minio, NGinx
* ```Dependencies (Basic + Kibana)``` will run the same containers as the Basic one but will add Kibana, filebeat and APM so you can have access to the kibana dashboard and debug your system more efficiently

#### Core components

The core components docker compose tasks run every single Assemblyline core components:

* Service server
* Frontend
* API Server
* Socket IO Server
* Alerter
* Expiry
* Metrics
* Heartbeats
* Statistics
* Workflow
* Plumber
* Dispatcher
* Ingester

!!! tip
    If you are wondering what each components does, you should read the [Core components](../../../core/components) which gives a brief description of every single one of them.

The core components tasks will also add two test user to the system:

| uid   | password | is_admin | apikey       |
| ----- | -------- | -------- | ------------ |
| user  | user     | no       | devkey:user  |
| admin | admin    | yes      | devkey:admin |

#### Scaler and Updater

This task is an extra core component task that will launch updater and scaler. This task requires you to run one of the denpendency task as well as the core components task to run properly. Scaler and updater have been seperated from the core components task because they interfere with running services live in the system. You are unlikely to ever run this task unless you work on a issue loading container from scaler or updater.

#### Services

This task will register all services to the system and allow them to be instanciated by scaler and updater. You are unlikely to run this task unless you want a working dev system that can scan files like an appliance can.

### Pytest dependencies

The pytest dependencies tasks are used to setup the environement properly to be able to run the tests like if your were building the packages. There are 4 possible dependencies for the tests:

* Base - To run the tests for the assemblyline-base repo
* Core - To run the tests for the assemblyline-core repo
* Service Server - To run the tests for the assemblyline-service-server repo
* UI or All - To run the tests for the assemblyline-ui repo or any other repos for that matter

!!! tip
    In most case, if you are running tests you can use ```UI or All``` tasks because all the tests will work with it but you might want to use the other tasks to save on resources and on dependency loading time.

## Launching components in debug mode

Using debug mode on all of Assemblyline's component will probably be the single most useful thing that is pre-configured in by the [setup script](../setup_script). All of Assemblyline's component get a launch target out of the box to simplify your debugging needs. Simply click the ```Run/Debug``` quick access side menu, select the component that you want to launch and click the play button.

![Launch targets](./images/launch.png)

The configured launch targets have been pre-fixed with category to help you identify what they do:

* Core - Core components unrelated to services
* Data - Script that generate random data in the system
* Service Server - Service API server
* Service - Service Live debugging
* UI - Assemblyline API Servers

!!! tip
    If you are wondering what each components does, you should read the [Core components](../../../core/components) which gives a brief description of every single one of them.

!!! warning
    To be able to successfully run these components, you will have to run at minimum the [Dependencies (Basic)](#dependencies) tasks

### Core, Service Server and UI

TBD

### CLI

TBD

### Data components

TBD

### Running live services

TBD
