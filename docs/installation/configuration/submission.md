# Submission section

The submission configuration section (`submission:`) of the configuration file allows you to modify the parameters off how submissions are handled in the system.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Submission section configuration example"
    ```yaml
    ...
    submission:
      # Maximum amount of extracted files for a submission
      default_max_extracted: 500

      # Maximum amount of supplementary files for a submission
      default_max_supplementary: 500

      # Default amount of days submissions live in the system
      dtl: 30

      # Maximum amount of days submissions live in the system
      max_dtl: 0

      # Maximum extraction depth service can go
      max_extraction_depth: 6

      # Maximum file size allowed in the system
      max_file_size: 104857600

      # Maximum size of each metadata entry
      max_metadata_length: 4096

      # Types of tags to be included in the submission summary in
      # the attribution, behaviour and ioc sectiona.
      tag_types:
        attribution:
        - attribution.actor
        - attribution.campaign
        - attribution.exploit
        - attribution.implant
        - attribution.family
        - attribution.network
        - av.virus_name
        - file.config
        - technique.obfuscation
        behavior:
        - file.behavior
        ioc:
        - network.email.address
        - network.static.ip
        - network.static.domain
        - network.static.uri
        - network.dynamic.ip
        - network.dynamic.domain
        - network.dynamic.uri
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
