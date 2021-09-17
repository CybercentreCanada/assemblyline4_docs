# Logging section

The logging configuration section (`logging:`) of the configuration file allows you to modify the log level and where the logs will be shipped in the system .

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Logging section configuration example"
    ```yaml
    ...
    logging:
      # Interval at which the container heartbeat is written
      export_interval: 5

      # Location of the container heartbeat
      heartbeat_file: /tmp/heartbeat

      # Should logs use a JSON format
      # (mainly used to parse logs into kibana, otherwise set to false to make them readable)
      log_as_json: true

      # Location on disk where the logs are stored if log_to_file enabled
      log_directory: /var/log/assemblyline/

      # Minimum log level
      # (DEBUG, INFO, WARNING, ERROR)
      log_level: INFO

      # Should logs be shown in the console?
      # You should have that to true if running inside containers
      log_to_console: true

      # Should you write logs to files?
      # Set this to false when running inside a container
      log_to_file: false

      # Should you send logs to a syslog server?
      log_to_syslog: false

      # Host of the syslog server
      syslog_host: localhost

      # Port of the syslog server
      syslog_port: 514
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
