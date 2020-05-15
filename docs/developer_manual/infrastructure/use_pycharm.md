---
layout: default
title: Use Pycharm
parent: Infrastructure
grand_parent: Developer's manual
has_children: false
has_toc: false
nav_order: 4
---

# Use Pycharm
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Here are some pointers on how to run most of the core components live in Pycharm when either your local or remote development VM is ready.

## Load docker-compose files

### Minimal Dependencies

When dependencies are loaded, you can launch any core components and its dependencies should be satisfied. 

#### Load dependencies the Pycharm Pro way

1. In your project file viewer
2. Browse to **assemblyline-base/dev/depends**
3. Right click on **docker-compose-minimal.yml**
4. Select **Create 'depends/docker-compose...'**
5. Click **OK**
6. You'll notice on the top right that a new Compose deployment has been created for dependencies, hit the play button next to it to launch the containers. 

From now on you can just select that Compose deployment for dependencies from the dropdown up top and hit the run button to launch it. It is good practice to Edit the configurations and give it a proper name.

#### Load dependencies the docker-compose way

In a new terminal on the VM, run the following commands:

    cd ~/git/alv4/assemblyline-base/dev/depends/
    sudo docker-compose -f docker-compose-minimal.yml up
    
This will take over the terminal and the logs will be displayed there. Hit **ctrl-c** when you want to shut the containers down.

If you close the terminal and the containers keep running, run the following commands to shut down the containers:

    cd ~/git/alv4/assemblyline-base/dev/depends/
    sudo docker-compose -f docker-compose-minimal.yml down

### Core services 

Core services depend on the dependencies docker-compose file. When the core services are loaded you should be able to point a browser at **https://IP_OF_VM** and you'll be greeted with a working version of assemblyline with no services configured. 

*Default admin user credentials are: **admin** / **admin***

#### Load core services the Pycharm Pro way

1. In your project file viewer
2. Browse to **assemblyline-base/dev/core**
3. Right click on **docker-compose.yml**
4. Select **Create 'core: Compose Deploy...'**
5. Click **OK**
6. You'll notice on the top right that a new Compose deployment has been created for core services, hit the play button next to it to launch the containers.

#### Load core services the docker-compose way

In a new terminal on the VM, run the following commands:

    cd ~/git/alv4/assemblyline-base/dev/core/
    sudo docker-compose up
    
This will take over the terminal and the logs will be displayed there. Hit **ctrl-c** when you want to shut the containers down.

If you close the terminal and the containers keep running, run the following commands to shut down the containers:

    cd ~/git/alv4/assemblyline-base/dev/core/
    sudo docker-compose down

## Run components live from pycharm

### Easy to run components (UI as an exemple)

Most of the core components are going to be as simple as this to run, find the component main file then hit Run or Debug... All core components require you to run the **Minimal Dependencies** docker-compose file prior to launch them.

 1. Find the UI main file in the project files browser
    - **assemblyline-ui/assemblyline_ui/app.py**
 2. Right click on it
 3. Select either **Run 'app'** or **Debug 'app'**

### Complex component (Debugging services live)

The main exception to very easy to run components are debugging services live in the system. Service require you to run two seperate components to process files. The two components talk via named pipes in the /tmp folder. Inside the docker container this is very easy but debugging this live needs requires some sacrifices.

 1. You can only run one at the time
 2. Depending on where you stop them and how you stop them, they don't fully cleanup after themselves.
 3. They require extra configuration, it's not just click and go

We will show you how to run the ResultSample service live in the system. This requires you to run the full infrastructure using the docker-compose files prior to execute the steps. (**Minimal dependencies** and **Core services**)

 1. In a shell on the **VM**,  make sure that there are no **service_manifest.yml** file in the **/tmp** folder: `rm /tmp/service_manifest.yml`
 2. From the project files browser find task_handler.py
    - **assemblyline-service-client/assemblyline_service_client/task_handler.py**
 3. Right click on it, then hit **Run** or **Debug**
    - Task handler is now waiting that the service_manifest.yml appears in the /tmp folder to tell it which service is running and how to interface to it
 4. Near the top right, there is a dropdown menu which now that you are running task handler should say: **task_handler**. Click on it and click **Edit configurations...**
 5. Click the **+** button at the top left
 6. Click **Python**
 7. First option in the configuration tab is a drop down that says **Script path:** click it and choose **Module Name:**
 8. In the box beside it, write: `assemblyline_v4_service.run_service`
 9. Add `;SERVICE_PATH=assemblyline_result_sample_service.result_sample.ResultSample` to the **Environment variables**
 10. Set the working directory to: `assemblyline-v4-service/assemblyline_result_sample_service` by pressing the little **folder** on the right and browsing to that directory
 11. In the name box at the top, write `ResultSample`
 12. Click **OK**
 13. Now that dropdown near the top says **ResultSample**, click the **play** or the **bug** button to either **Run** or **Debug** the service 
    - In the Run or Debug window, the service will be stuck at *Waiting for receive task named pipe to be ready...*. This is because task_handler shutdown after registering the service for the first time.
 14. Select **task_handler** from the dropdown near the top and click the **play** or **bug** button beside it again.
    - Now both ResultSample and task_handler will stay up an poll for task from service server.
