# *ServiceUpdater* class
Some services created for Assemblyline will require signatures/rules as part of it's analysis process. `ServiceUpdater` class which can be imported from `assemblyline_v4_service.updater.updater`. In this section we will go through the different methods and variables available to you in the `ServiceUpdater` class.


You can view the source for the class here: [ServiceUpdater class source](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/updater/updater.py)

## Class variables
The `ServiceUpdater` base class includes many instance variables which can be used to access service related information.

The following tables describes all of the variables of the `ServiceUpdater` class.

| Variable Name | Description |
|:---|:---|
| config | Reference to the service parameters containing values updated by the user for service configuration. |
| dependencies | A dictionary containing connection details for service dependencies|
| log | Reference to the logger. |
| service_attributes | Service attributes from the [service manifest](../service_manifest). |
| rules_directory | Returns the directory path which contains the current location of your rules |
| rules_hash | A hash of the files in the rules_list. Used to invalidate caching if rules change.|
| rules_list | Returns a list of directory paths which point to rule files derived from rules_directory|
| update_time | An integer representing the epoch of when the last update occured|
| working_directory | Returns the directory path which the service can use to temporarily store files during each task execution. |



## Class functions
This is the list of all the functions that you can override in your updater. They are explain in order of important and the likelihood at which you will override them.

Note: the updater are yours to define however you would like for your service, we have implemented the basics that work with our existing services **but** this does not mean you have to follow what's already defined. Override it!
ie. [Safelist] (https://github.com/CybercentreCanada/assemblyline-service-safelist)

### _load_rules()
The `_load_rules` function is called to process the rules_list in a specific way defined by the service

### _clear_rules()
The `_clear_rules` function is optionally called to remove the current ruleset from memory. Requires implementation by the service writer.

### _download_rules()
The `_download_rules` function is called after each `_cleanup` call to check if there is new updates to be processed. If so, it will attempt to download and use the new ruleset otherwise it will revert to the old ruleset.

It will call on `_load_rules` and `_clear_rules` during this attempt process.

### do_local_update()
The `do_local_update` function is called on a separate thread that checks Assemblyline for local changes to signatures such as change in status or additions/removals to the ruleset. This will then fetch the new ruleset on modification and make it available to be served to service instances.

### do_source_update()
The `do_source_update` function is called on a separate thread that checks external signature sources for changes. This will then fetch the new ruleset on modification and update the signature set in Assemblyline to make it available to both users and the `do_local_update` thread.
