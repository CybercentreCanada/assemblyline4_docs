# Configuration YAML file

Assemblyline 4 configuration is done using a YAML file (`config.yml`) which is deployed to all containers when they are launched.

## Specification and defaults

The full specification of the file [is defined here](https://github.com/CybercentreCanada/assemblyline-base/blob/master/assemblyline/odm/models/config.py). The Object Data Model (ODM) converts the python model to a YAML file which looks like the following by default:

??? info "Default configuration file values"
    ```YAML
    auth:
      allow_2fa: true
      allow_apikeys: true
      allow_extended_apikeys: true
      allow_security_tokens: true
      internal:
        enabled: true
        failure_ttl: 60
        max_failures: 5
        password_requirements:
          lower: false
          min_length: 12
          number: false
          special: false
          upper: false
        signup:
          enabled: false
          notify:
            activated_template: null
            api_key: null
            authorization_template: null
            base_url: null
            password_reset_template: null
            registration_template: null
          smtp:
            from_adr: null
            host: null
            password: null
            port: 587
            tls: true
            user: null
          valid_email_patterns:
          - .*
          - .*@localhost
      ldap:
        admin_dn: null
        auto_create: true
        auto_sync: true
        base: ou=people,dc=assemblyline,dc=local
        bind_pass: null
        bind_user: null
        classification_mappings: {}
        email_field: mail
        enabled: false
        group_lookup_query: (&(objectClass=Group)(member=%s))
        image_field: jpegPhoto
        image_format: jpeg
        name_field: cn
        signature_importer_dn: null
        signature_manager_dn: null
        uid_field: uid
        uri: ldap://localhost:389
      oauth:
        enabled: false
        gravatar_enabled: true
        providers:
          auth0:
            access_token_url: https://{TENANT}.auth0.com/oauth/token
            api_base_url: https://{TENANT}.auth0.com/
            authorize_url: https://{TENANT}.auth0.com/authorize
            client_id: null
            client_kwargs:
              scope: openid email profile
            client_secret: null
            jwks_uri: https://{TENANT}.auth0.com/.well-known/jwks.json
            user_get: userinfo
          azure_ad:
            access_token_url: https://login.microsoftonline.com/common/oauth2/token
            api_base_url: https://login.microsoft.com/common/
            authorize_url: https://login.microsoftonline.com/common/oauth2/authorize
            client_id: null
            client_kwargs:
              scope: openid email profile
            client_secret: null
            jwks_uri: https://login.microsoftonline.com/common/discovery/v2.0/keys
            user_get: openid/userinfo
          google:
            access_token_url: https://oauth2.googleapis.com/token
            api_base_url: https://openidconnect.googleapis.com/
            authorize_url: https://accounts.google.com/o/oauth2/v2/auth
            client_id: null
            client_kwargs:
              scope: openid email profile
            client_secret: null
            jwks_uri: https://www.googleapis.com/oauth2/v3/certs
            user_get: v1/userinfo

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

    datasources:
      al:
        classpath: assemblyline.datasource.al.AL
        config: {}
      alert:
        classpath: assemblyline.datasource.alert.Alert
        config: {}

    datastore:
      hosts:
      - http://elastic:devpass@localhost
      ilm:
        days_until_archive: 15
        enabled: false
        indexes:
          alert: &id002
            cold: 15
            delete: 30
            unit: d
            warm: 5
          error: *id002
          file: *id002
          result: *id002
          submission: *id002
        update_archive: false
      type: elasticsearch

    filestore:
      cache:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-cache&use_ssl=False
      storage:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-storage&use_ssl=False

    logging:
      export_interval: 5
      heartbeat_file: /tmp/heartbeat
      log_as_json: true
      log_directory: /var/log/assemblyline/
      log_level: INFO
      log_to_console: true
      log_to_file: false
      log_to_syslog: false
      syslog_host: localhost
      syslog_port: 514

    services:
      allow_insecure_registry: false
      categories:
      - Antivirus
      - Dynamic Analysis
      - External
      - Extraction
      - Filtering
      - Internet Connected
      - Networking
      - Static Analysis
      cpu_reservation: 0.25
      default_timeout: 60
      image_variables: {}
      min_service_workers: 0
      preferred_update_channel: stable
      stages:
      - FILTER
      - EXTRACT
      - CORE
      - SECONDARY
      - POST
      - REVIEW

    submission:
      default_max_extracted: 500
      default_max_supplementary: 500
      dtl: 30
      max_dtl: 0
      max_extraction_depth: 6
      max_file_size: 104857600
      max_metadata_length: 4096
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

    system:
      constants: assemblyline.common.constants
      organisation: ACME
      type: production

    ui:
      allow_malicious_hinting: false
      allow_raw_downloads: true
      allow_url_submissions: true
      audit: true
      banner: null
      banner_level: info
      debug: false
      download_encoding: cart
      email: null
      enforce_quota: true
      fqdn: localhost
      ingest_max_priority: 250
      read_only: false
      read_only_offset: ''
      secret_key: This is the default flask secret key... you should change this!
      session_duration: 3600
      statistics:
        alert:
        - al.attrib
        - al.av
        - al.behavior
        - al.domain
        - al.ip
        - al.yara
        - file.name
        - file.md5
        - owner
        submission:
        - params.submitter
      tos: null
      tos_lockout: false
      tos_lockout_notify: null
      url_submission_headers: {}
      url_submission_proxies: {}
      validate_session_ip: true
      validate_session_useragent: true
    ```

## Layers of the configuration file

The configuration file is built in layers:

1. The ODM converts the python classes to the default values as shown above
2. The default [assemblyline helm chart values.yaml file](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/eeceda628fa7a7ab23bda3ab3132e7dc61bd6c95/assemblyline/values.yaml#L223) changes certain of these values to adapt them to a Kubernetes deployment
3. Your deployment's `values.yaml` file change the values to their final form

## Changing the configuration file

If you want to change the `config.yml` file that will be deployed in the containers, it will have to be done through the `configuration` section found in the `values.yml` file of your deployment.

!!! example
    Let's say that you would want to change the log level in the system to `ERROR` an up.

    First of you would edit the `values.yaml` file of your personal deployment to add the changes to the configuration section:
    ```YAML
    ...
    configuration:
      logging:
        log_level: ERROR
    ...
    ```

    Then you would simply deploy that new `values.yaml` file using the `helm upgrade` command specific to your deployment:

    * [Cluster deployment update](../../cluster/#update-your-deployment)
    * [Appliance deployment update](../../appliance/#updating-the-current-deployment)

## Exhaustive configuration file documentation

All parameters of each configuration section will be thoroughly documented in their respective pages.

Here are the links to the different section documentations:

* [Authentication (auth:)](../authentication)
* [Core components  (core:)](../core)
* [Data sources (datasources:)](../datasources)
* [Database (datastore:)](../datastore)
* [File storage (filestore:)](../filestore)
* [Logging (logging:)](../logging)
* [Services (services:)](../services)
* [Submission (submission:)](../submission)
* [System (system:)](../system)
* [User Interface (ui:)](../ui)
