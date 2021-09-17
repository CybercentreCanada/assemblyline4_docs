# Data sources section

The Data Sources configuration section (`datasources:`) of the configuration file contains all the different parameters that you can change to add/modify data sources used by the hash search API.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Datasources section configuration example"
    ```yaml
    ...
    # The data source section is essentially a key/value pair of source and their configuration
    datasources:
      # Source name (al)
      al:
        # Path to the module that will process the query received by the hash search API
        classpath: assemblyline.datasource.al.AL

        # Dictionary holding the configuration for the module
        config: {}

      # Source name (alert)
      alert:
        # Path to the module that will process the query received by the hash search API
        classpath: assemblyline.datasource.alert.Alert

        # Dictionary holding the configuration for the module
        config: {}
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
