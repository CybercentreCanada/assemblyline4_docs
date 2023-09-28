# Service section

The service configuration section (`services:`) of the configuration file allows you to modify the parameters on how services are executed in the system.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Service section configuration example"
    ```yaml
    ...
    services:
      # Are we allowed to pull service containers from insecure registries
      allow_insecure_registry: false

      # List of valid categories for services
      # You should not change this except to add a category maybe?
      categories:
      - Antivirus
      - Dynamic Analysis
      - External
      - Extraction
      - Filtering
      - Internet Connected
      - Networking
      - Static Analysis

      # Percentage of CPU resevation scaler will do for each service (1 = 100%)
      cpu_reservation: 0.25

      # Default service execution timeout
      default_timeout: 60

      # Set of environment varables applied to the service containers
      # while loaded from updater or scaler
      image_variables: {}

      # Type of services the updater will be looking for when looking for service update
      # (dev or stable)
      preferred_update_channel: stable

      # List of available
      stages:
      - FILTER
      - EXTRACT
      - CORE
      - SECONDARY
      - POST
      - REVIEW
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
