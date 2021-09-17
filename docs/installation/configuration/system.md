# System section

The system configuration section (`system:`) of the configuration file allows you to modify the parameters of your system.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "System section configuration example"
    ```yaml
    ...
    system:
      # Path path to the constants module
      constants: assemblyline.common.constants

      # Organisation Name
      organisation: ACME

      # Type of system. If different then production, watermark will be shown in the UI
      type: production
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
