# Core component section

The core components configuration section (`core:`) of the configuration file contains all the different parameters that you can change to modify the behaviour of each core components.

??? info "Default values for the core section"
    ```yaml
    ...
    core:
      alerter:
        alert_ttl: 90
        constant_alert_fields:
        - alert_id
        - file
        - ts
        default_group_field: file.sha256
        delay: 300
        filtering_group_fields:
        - file.name
        - status
        - priority
        non_filtering_group_fields:
        - file.md5
        - file.sha1
        - file.sha256
        process_alert_message: assemblyline_core.alerter.processing.process_alert_message
      dispatcher:
        max_inflight: 1000
        timeout: 900
      expiry:
        batch_delete: false
        delay: 0
        delete_storage: true
        sleep_time: 15
        workers: 20
      ingester:
        cache_dtl: 2
        default_max_extracted: 100
        default_max_supplementary: 100
        default_resubmit_services: []
        default_services: []
        default_user: internal
        description_prefix: Bulk
        expire_after: 1296000
        get_whitelist_verdict: assemblyline.common.signaturing.drop
        incomplete_expire_after_seconds: 3600
        incomplete_stale_after_seconds: 1800
        is_low_priority: assemblyline.common.null.always_false
        max_inflight: 500
        sampling_at:
          critical: 500000
          high: 1000000
          low: 10000000
          medium: 2000000
        stale_after_seconds: 86400
        whitelist: assemblyline.common.null.whitelist
      metrics:
        apm_server:
          server_url: null
          token: null
        elasticsearch:
          cold: 30
          delete: 90
          host_certificates: null
          hosts: null
          unit: d
          warm: 2
        export_interval: 5
        redis: &id001
          host: 127.0.0.1
          port: 6379
      redis:
        nonpersistent: *id001
        persistent:
          host: 127.0.0.1
          port: 6380
      scaler:
        additional_labels: null
        cpu_overallocation: 1
        memory_overallocation: 1
        overallocation_node_limit: null
        service_defaults:
          backlog: 100
          environment:
          - name: SERVICE_API_HOST
            value: http://service-server:5003
          - name: AL_SERVICE_TASK_LIMIT
            value: inf
          growth: 60
          min_instances: 0
          shrink: 30
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.

## Alerter

The configuration block at `core.alerter` contains all the configuration parameters that the Assemblyline alerter component can take.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Alerter configuration example"
    ```yaml
    core:
      alerter:
        # Time to live in days for alerts in the system
        alert_ttl: 90

        # List of fields that should keep the same value in between normal and extended scan
        #  NOTE: You should not have to change those ever, this might get removed in the future
        constant_alert_fields:
        - alert_id
        - file
        - ts

        # Default field to group alerts with in the UI
        default_group_field: file.sha256

        # Delay applied to the alert UI to leave time to the extended scan to complete
        delay: 300

        # List of fields allowed to be used for grouping that are present in every single alerts
        filtering_group_fields:
        - file.name
        - status
        - priority

        # List of fields allowed to be used for grouping that are present only in a subset of alerts
        non_filtering_group_fields:
        - file.md5
        - file.sha1
        - file.sha256

        # Python class used to process alert messages
        #   NOTE: This should not be changed unless you've built your own container with
        #         extra packages with alert processing capability
        process_alert_message: assemblyline_core.alerter.processing.process_alert_message
    ```

## Dispatcher

The configuration block at `core.dispatcher` contains all the configuration parameters that the Assemblyline dispatcher component can take.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Dispatcher configuration example"
    ```yaml
    core:
      dispatcher:
        # Maximum amount of concurrent submission processing in the system
        max_inflight: 1000

        # Use in earlier version of dispatcher (To be removed)
        timeout: 900
    ```

## Expiry

The configuration block at `core.expiry` contains all the configuration parameters that the Assemblyline expiry component can take.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Expiry configuration example"
    ```yaml
    core:
        expiry:
          # Perform delete operation in batch when changing day instead of throughout the day
          #   NOTE: Not deleting during the day will make the system more responsive during the day
          #         but will significantly reduce performance when changing to a new day until
          #         all delete operations are done.
          batch_delete: false

          # Delay in hours applied to the deletion schedule
          delay: 0

          # Should data expiry operation get rid of files as well?
          delete_storage: true

          # Time to sleep (sec) in between runs when there is not data to delete
          sleep_time: 15

          # Number of workers used to delete data
          workers: 20
    ```

## Ingester

The configuration block at `core.ingester` contains all the configuration parameters that the Assemblyline ingester component can take.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Ingester configuration example"
    ```yaml
    core:
        ingester:
          # Number of days ingested files are valid in the ingester cache
          cache_dtl: 2

          # Maximum number of extracted files for ingested submissions
          default_max_extracted: 100

          # Maximum number of supplementary files for ingested submissions
          default_max_supplementary: 100

          # UNUSED
          default_resubmit_services: []
          default_services: []
          default_user: internal
          description_prefix: Bulk

          # Seconds before a previously ingested file will be considered new again,
          #  the file will then be processed as if never seen before.
          expire_after: 1296000

          # Function import path for method to determine whitelisting.
          #  Files selected by this function will be dropped.
          #  See the default function for signature.
          get_whitelist_verdict: assemblyline.common.signaturing.drop

          # Special version of 'expire_after' applied when the previous run of a file had errors.
          incomplete_expire_after_seconds: 3600

          # Special version of 'stale_after_seconds' applied when the previous run of a file had errors.
          incomplete_stale_after_seconds: 1800

          # Function import path for method to determine low-priority filter.
          #  Files selected by this function will be forced to low-priority.
          #  See the default function for signature.
          is_low_priority: assemblyline.common.null.always_false

          # How many submissions should ingester try to submit concurrently.
          max_inflight: 500

          # How long should X queue be before the sampling (randomly dropping files) starts.
          #  At the given value sampling will start, and grow gradually more agressive until 3 times
          #  the value given. At 3 times the value given the system will proccess as many files
          #  as possible, and all others will be discarded.
          sampling_at:
            critical: 500000
            high: 1000000
            low: 10000000
            medium: 2000000

          # Seconds before a previously ingested file will be considered stale,
          #  the file will be reprocessed but the priority will be modified depending
          #  on the score from the previous run.
          stale_after_seconds: 86400

          # File whitelist import path. Imported object will be passed to the 'get_whitelist_verdict'
          #  function. The default verdict function will use this as a file whitelist.
          #  See default value for sample.
          whitelist: assemblyline.common.null.whitelist
    ```

## Metrics

The configuration block at `core.metrics` contains all the configuration parameters that the Assemblyline metrics gathering components can take.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Metrics configuration example"
    ```yaml
    core:
        metrics:
          # APM specific settings
          apm_server:
            # URL to the APM server
            server_url: null

            # Token use to connect to the APM server
            token: null

          # Elasticsearch specific configuration
          elasticsearch:
            # Number of `unit` document spend time in ILM cold storage
            cold: 30

            # Number of `unit` after which documents are deleted using ILM
            delete: 90

            # Cert used to connect to Elastic
            host_certificates: null

            # List of Elatic hosts
            hosts: null

            # Time unit for ILM cold, warm and delete
            unit: d

            # Number of `unit` document spend time in ILM warm storage
            warm: 2

          # Interval at which the metrics components export their data
          export_interval: 5

          # Redis specific configuration
          redis:
            # Host of the redis server
            host: 127.0.0.1

            # Port of the redis server
            port: 6379
    ```

## Redis

The configuration block at `core.redis` contains all the configuration parameters used by Assemblyline components to connect to redis.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Redis configuration example"
    ```yaml
    core:
        redis:
          # Configuration of the non-persistent redis (use mainly for messaging)
          nonpersistent:
            # Host of the redis server
            host: 127.0.0.1

            # Port of the redis server
            port: 6379

          # Configuration of the persistent redis (use mainly for task queuing)
          persistent:
            # Host of the redis server
            host: 127.0.0.1

            # Port of the redis server
            port: 6380
    ```

## Scaler

The configuration block at `core.scaler` contains all the configuration parameters that the Assemblyline scaler component can take.

Here is an example configuration block with inline comments about the purpose of every single parameters:

???+ example "Scaler configuration example"
    ```yaml
    core:
        scaler:
          # Percentage of CPU overallocation, represented as 0-1 float
          cpu_overallocation: 0.5
          # Percentage of RAM overallocation, represented as 0-1 float
          memory_overallocation: 1
          # If the system has this many nodes or more overallocation is ignored
          overallocation_node_limit: 3
          # Additional labels to be applied to services('=' delimited)
          additional_labels:
            - env=production
          service_defaults:
            # How many files in a service queue are considered a backlog.
            #  This weights how important scaling up a service is relative
            #  to its queue. You probably don't want to change this.
            backlog: 100

            # List of environment variables set for all services.
            #  Usually used for deployment related environment variables.
            #  Service specific environent variables can be set in the
            #  service manifest or in the UI for a particular system.
            environment:
            - name: SERVICE_API_HOST
              value: http://service-server:5003
            - name: AL_SERVICE_TASK_LIMIT
              value: inf

            # Roughly how many seconds to wait before a service
            #  scales up to meet increased demand.
            growth: 60

            # The minimum number of instances of every service that
            #  should be kept running at all times.
            min_instances: 0

            # Roughly how many seconds to wait before a service scales down when
            #  instances are consistently idle.
            shrink: 30
    ```
