# Service management

Assemblyline's service management interface lets you:

1. List all the services in the system
3. View details about those services
4. Add/Modify/Remove services
5. Download/Restore a backup of the current services configurations

You can find the service management interface by clicking the *Administration* topic then choose *Services* subtopic.

![Service management](./images/services_bar.png)

## Service list

The first page you will be taken to when loading the service management interface will list all the services of the system.

![Service list](./images/service_list.png)

This page is important so let's talk about everything, starting with the buttons on the top right of the page:

![Service management buttons](./images/service_mgmt_buttons.png)


These buttons perform the following:

1. Add services to the system
2. Perform service updates
3. Install all available services
4. Download a backup of the current services configurations
5. Restore services configuration from a backup

## Add a service

You can add a service by clicking the circled green "*(+)*" sign in the top right corner. This will open a popup window with an empty textbox.

![Service add](./images/service_add.png)

Simply paste the `service_manifest.yml` content of the service you which to add to the system then hit the "*Add*" button to add it to the system.

!!! tip
    If your manifest uses the following environment variables, they will be replaced by the right values by the service add API:

    * **$SERVICE_TAG**: Will de replaced by the latest tag for your current deployment type (dev/stable) found in the docker registry where the service container is hosted
    * **$REGISTRY**: Will be replaced by your local registry

### Update services

If the system detected that there is a container with a newer version for your current deployment type (dev/stable). The service list will show an update button.

Development on Assemblyline's supported services is rapid, and releases are cut quite frequently to keep up with malware development. Therefore, this button is helpful to action updating all services that have updates available.

If you want to update a single service, a button will appear on that service card indicating that a newer version is available.

![Service update](./images/service_update.png)

Hovering over the button will let you know which new service version is available and clicking the button will kick off the update for the service.

## Install Available Services

The next button after that is "Install all services". If you scroll to the bottom of the page, you can see the services that you do not have installed but are available for installation. This button will install all the available services, if any, or you can install each one manually by scrolling down to the section and clicking on each card's button.

![Services available section](./images/services_available.png)

## Create / Restore Service config backups

At the top right corner of the service management page, you will also find backup and restore buttons for creating and restoring backups for the services configurations.

The backup button which looks like an "*arrow pointing down*" will create a yml with a filename of the following format: `<FQDN>_service_backup.yml`. The file will automatically be downloaded by your browser in your download directory.

Once you want to restore the backup in your system, you can simply click the restore button, "*clock with a counter clockwise arrow*", This will open a modal window with an empty textbox.

![Service restore](./images/service_restore.png)

Simply paste the content of the backup created earlier in the text box and hit the "*Restore*" button to restore the services configurations to their backed-up values.

## Service Listing Overview

Let's look at the columns and data in the service table:

![Service Management page](./images/service_mgmt_page.png)

The "Name" column contains the service name, and the "Version" column contains the version of the service that is running on Assemblyline. 

The "Category" column refers to a service's category, which is a way of grouping services. Available categories out-of-the-box are Antivirus, Dynamic Analysis, External, Extraction, Filtering, Internet Connected, Networking, and Static Analysis. 

The "Stage" column defines when a service should run. Possible stages are FILTER, EXTRACT, CORE, SECONDARY, POST and REVIEW. Stages are executed in the order defined in the list.

"File types" contains a regular expression that dictates which Assemblyline file types a service will accept. If you want all files to be analyzed by a service, then specify the following regular expression .* as seen in the AVClass and Ancestry services above.

The "External" column indicates if a service will send a file or related data somewhere outside of Assemblyline's infrastructure. An example of this is the VirusTotal service, which will send a file to the VirusTotal platform for analysis.

The "Mode" column specifies if a service "runs in privileged mode" or "uses service server". This configuration is explained in the Assemblyline documentation here. 

The "Classification" column is the classification level that the service operates at and will label its results as such.

Finally, the "Enabled" column indicates if the service is enabled or disabled. If a service is disabled, there will be zero pods running that service, and the service will not accept files.

## Service Details

If you wish to modify or remove a service, you can simply click on that service from the service list which will bring you to the service detail page.

The service detail page header contains two buttons shown all time that will let you:

* Delete the service (red "*circled minus*" button)
* Toggle between enabled/disable state (big square button right on top of the tabs)

You will then have a tabbed interface which we will describe each tab bellow.

### General tab

The "*General*" tab will let you see general information about the service.

![Service detail general](./images/service_detail_gen.png)

In this tab, you will be able to modify the service's:

* Version
* Description
* Execution Stage
* Category
* Accepted/Rejected file types
* Execution timeout
* Maximum number of instances
* Location
* Result caching

!!! tip
    You can refer to the [service manifest](../../developer_manual/services/advanced/service_manifest/) documentation for more information about those different fields.

### Container tab

The "*Container*" tab will show information about containers used by the service.

![Service detail container](./images/service_detail_container.png)

In this tab, you will be able to:

* Change the update channel (Development/Stable)
* Change the main service container
* Add/modify/remove dependency containers

#### Main service container

The main service container is the container containing and running the service code. By clicking the main service container, you will be able to modify the parameters used to launch that container.

![Service detail container edit](./images/service_detail_container_edit.png)

The list of parameters you will be able to modify is the following:

* Container image name
* Type of container registry
* Resources limits (CPU/RAM)
* Container registry credentials (username/password)
* Command executed in the container
* Allow internet access to the container
* Environment variables set before loading the container

!!! tip
    Check the [docker config](../../developer_manual/services/advanced/service_manifest/#docker-config) block from the [service manifest](../../developer_manual/services/advanced/service_manifest/) documentation to know more about the different field you can modify in the docker container configuration.

#### Dependency containers

Dependency containers are containers use to support the main services in some ways. Either by offering an external place to store data (A database for example) or to perform service updates.

A service can have multiple dependency containers and these containers are shared between the multiple instances of the service that can be loaded in the system i.e., there will only be one instance of each dependency container.

By either click the "*Add Dependency*" button or clicking a dependency container, you will be able to either add or modify container dependencies of the current service.

![Service detail dependency edit](./images/service_detail_dependency_edit.png)

The dependency container configuration window look almost the same and let you modify the same values as the [main service container](#main-service-container) window. There is however an added parameter that you can configure to give the container persistent storage.

!!! tip
    Check the [persistent volume](../../developer_manual/services/advanced/service_manifest/#persistent-volume) block from the [service manifest](../../developer_manual/services/advanced/service_manifest/) documentation to know more about the different fields to configure to get persistent storage in a dependency container.

### Updates tab

The "*Updates*" tab shows information about how the service updates itself or its signatures.

![Service detail updates](./images/service_detail_updates.png)

!!! warning
    This tab is optional and will not be shown for all service. Only services that define and [update config](../../developer_manual/services/advanced/service_manifest/#update-config) block in their [service manifest](../../developer_manual/services/advanced/service_manifest/) will have that tab shown.

In this tab, you will be able to view/modify the following information:

* Interval at which the service updates
* If the service generates signatures in the system or not
* If the service needs to wait for a successful update to start instances of itself
* The various sources where the service pulls its updates from

!!! tip
    Checkout the [Modifying sources](../../administration/source_management/#modifying-sources) documentation to know more about the different values you can change in the signature sources.

### Parameters tab

Finally, the "*Parameters*" tab will let you view and customize the different parameters the service can take in.

![Service detail parameters](./images/service_detail_parameters.png)

Service parameters are split into two categories:

* User specified parameters
* Service variables

#### User specified parameters

User specified parameters are parameters that a user can modify for each specific submission it does in the system.

They are often but not exclusively used for things like:

* Turning on/off features of a service
* Specifying a password used during a submission
* Limit what the service can and cannot do
* Extract more or less files when a service runs

!!! tip
    When these parameters are defined for a service, they will be shown in the [submission options](../../user_manual/submitting_file/#options) available for the user at submission time.

#### Service variables

Service variable are configuration parameters only shared between the service and your deployment. They are used to help the service configure itself to run well in your environment.

Service variables are often but not exclusively things like:

* URLs to connect to external services
* Credentials use to connect to external services
* List of default values used in a service
* Configuration parameter that will limit or increase scanning capabilities of a service
