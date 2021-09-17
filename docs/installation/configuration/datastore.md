# Database section

The Database configuration section (`datastore:`) of the configuration file contains all the different parameters that you can change modify how to connect to the database and to modify the Index Lifecycle Management (ILM) parameters.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Datastore section configuration example"
    ```yaml
    ...
    datastore:
      # List of elastic hosts to connect to
      hosts:
      - http://elastic:devpass@localhost

      # Index Lifecycle management configuration block
      ilm:
        # After how many days do documents go in the ILM managed indexes
        days_until_archive: 15

        # Is ILM enabled or not?
        enabled: false

        # Index specific ILM configuration
        indexes:
          alert:
            # After how many `unit` documents goes in cold storage
            cold: 15

            # After how many `unit` documents are deleted
            delete: 30

            # Time unit definition for the current index
            unit: d

            # After how many `unit` documents goes in warm storage
            warm: 5
          error:
            cold: 15
            delete: 30
            unit: d
            warm: 5
          file:
            cold: 15
            delete: 30
            unit: d
            warm: 5
          result:
            cold: 15
            delete: 30
            unit: d
            warm: 5
          submission:
            cold: 15
            delete: 30
            unit: d
            warm: 5

        # Show saving new document update it's archive counterpart
        #  NOTE: Setting this to false makes it faster but it will be possible to have
        #        duplicate documents
        update_archive: false

      # Type of datastore (only elasticsearch is supported so far, do not change)
      type: elasticsearch
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
