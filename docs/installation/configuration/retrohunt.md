# Retrohunt section

This section goes over how to configure the deployment instance to ensure the Retrohunt component is configured properly.

The Retrohunt section (`retrohunt:`) of the configuration file contains all the different parameters that you can change.

Refer to the [Retrohunt](../../overview/architecture.md/#yara-back-in-time-retrohunt) to get an understanding of how this feature works.

???+ example "Retrohunt configuration example"
    ```yaml
    ...
    retrohunt:
      # Is the Retrohunt functionnality enabled on the frontend
      enabled: false

      # Number of days retrohunt jobs will remain in the system by default
      dtl: 30

      # Maximum number of days retrohunt jobs will remain in the system
      max_dtl: 0

      # Base URL for service API
      url: https://hauntedhouse:4443

      # Service API Key
      api_key: ChangeThisDefaultRetroHuntAPIKey!

      # Should tls certificates be verified
      tls_verify: true
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more details on where and how to change the configuration of the system.
