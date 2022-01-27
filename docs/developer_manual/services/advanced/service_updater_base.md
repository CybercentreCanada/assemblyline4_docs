# *ServiceUpdater* class
Some services created for Assemblyline will require signatures/rules as part of it's analysis process. `ServiceUpdater` class which can be imported from `assemblyline_v4_service.updater.updater`. In this section we will go through the different methods and variables available to you in the `ServiceUpdater` class.


You can view the source for the class here: [ServiceUpdater class source](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/updater/updater.py)

## Class variables
The `ServiceUpdater` base class includes many instance variables which can be used to access service related information.

The following tables describes all of the variables of the `ServiceUpdater` class.

| Variable Name | Description |
|:---|:---|
| updater_type | The type of updater, typically the service's name lower-cased taken from the SERVICE_PATH environment variable|
| default_pattern | The default regex pattern used if a source doesn't provide one (Default: .*)|

## Class functions
This is the list of all the functions that you can override in your updater. They are explained in order of importance and the likelihood at which you will override them.

Note: the updater are yours to define however you would like for your service, we have implemented the basics that work with our existing services **but** this does not mean you have to follow what's already defined. Override it!
ie. [Safelist] (https://github.com/CybercentreCanada/assemblyline-service-safelist)

### import_update() [Implementation Required]
The `import_update` function is called to import a set of files into Assemblyline. This involves creating a list of `Signature` objects and importing them via the Signature API by using the Assemblyline client.

### do_source_update() [Override Optional]
The `do_source_update` function is called on a separate thread that checks external signature sources for changes. This will then fetch the new ruleset on modification and update the signature set in Assemblyline to make it available to both users and the `do_local_update` thread.

### is_valid() [Override Optional]
The `is_valid` function is called to determine if a file from a source is a valid file. The definition of its validity can vary between services but it should be able to answer the question 'Can the service use this?'.

### do_local_update() [Override Optional]
The `do_local_update` function is called on a separate thread that checks Assemblyline for local changes to signatures such as change in status or additions/removals to the ruleset. This will then fetch the new ruleset on modification and make it available to be served to service instances.


!!! warning
    Failure to implement import_update() in your service's subclass of `ServiceUpdater` will render the updater unusable.

For an example, see [Adding a Service Updater](../adding_a_service_updater.md)
