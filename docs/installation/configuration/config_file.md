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
      apikey_max_dtl: null
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
        auto_properties: []
        auto_sync: true
        base: ou=people,dc=assemblyline,dc=local
        bind_pass: null
        bind_user: null
        classification_mappings: {}
        email_field: mail
        enabled: false
        group_lookup_query: (&(objectClass=Group)(member=%s))
        group_lookup_with_uid: false
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
      saml:
        attributes:
          email_attribute: email
          fullname_attribute: name
          group_type_mapping: {}
          groups_attribute: groups
          roles_attribute: roles
        auto_create: true
        auto_sync: true
        enabled: false
        lowercase_urlencoding: false
        settings:
          debug: false
          idp:
            entity_id: https://mocksaml.com/api/saml/metadata
            single_sign_on_service:
              binding: urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect
              url: https://mocksaml.com/api/saml/sso
          sp:
            assertion_consumer_service:
              binding: urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST
              url: https://localhost/api/v4/auth/saml/acs/
            entity_id: https://assemblyline/sp
            name_id_format: urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified
          strict: false
    core:
      alerter:
        alert_ttl: 90
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
        threshold: 500
      archiver:
        alternate_dtl: 0
        minimum_required_services: []
        use_webhook: false
        webhook:
          ca_cert: null
          headers: []
          method: POST
          password: null
          proxy: null
          retries: 3
          ssl_ignore_errors: false
          uri: https://archiving-hook
          username: null
      dispatcher:
        max_inflight: 1000
        timeout: 900
      expiry:
        badlisted_tag_dtl: 0
        batch_delete: false
        delay: 0
        delete_batch_size: 2000
        delete_storage: true
        delete_workers: 2
        iteration_max_tasks: 50
        safelisted_tag_dtl: 0
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
      plumber:
        notification_queue_interval: 1800
        notification_queue_max_age: 86400
      redis:
        nonpersistent: *id001
        persistent:
          host: 127.0.0.1
          port: 6380
      scaler:
        additional_labels: null
        cpu_overallocation: 1
        linux_node_selector:
          field: []
          label: []
        memory_overallocation: 1
        overallocation_node_limit: null
        privileged_services_additional_labels: null
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
      updater:
        job_dockerconfig:
          cpu_cores: 1
          ram_mb: 1024
          ram_mb_min: 256
        registry_configs:
        - name: registry.hub.docker.com
          proxies: {}
    datasources:
      al:
        classpath: assemblyline.datasource.al.AL
        config: {}
      alert:
        classpath: assemblyline.datasource.alert.Alert
        config: {}
    datastore:
      archive:
        enabled: false
        indices:
        - file
        - submission
        - result
      cache_dtl: 5
      hosts:
      - http://elastic:devpass@localhost:9200
      type: elasticsearch
    filestore:
      archive:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-archive&use_ssl=False
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
    retrohunt:
      api_key: ChangeThisDefaultRetroHuntAPIKey!
      dtl: 30
      enabled: false
      max_dtl: 0
      tls_verify: true
      url: https://hauntedhouse:4443
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
      default_auto_update: false
      default_timeout: 60
      image_variables: {}
      preferred_update_channel: stable
      registries: []
      safelist:
        enabled: true
        enforce_safelist_service: false
        hash_types:
        - sha1
        - sha256
      stages:
      - FILTER
      - EXTRACT
      - CORE
      - SECONDARY
      - POST
      - REVIEW
      update_image_variables: {}
    submission:
      default_max_extracted: 500
      default_max_supplementary: 500
      default_temporary_keys:
        email_body: union
        passwords: union
      dtl: 30
      emptyresult_dtl: 5
      file_sources: []
      max_dtl: 0
      max_extraction_depth: 6
      max_file_size: 104857600
      max_metadata_length: 4096
      max_temp_data_length: 4096
      metadata:
        archive: {}
        ingest:
          INGEST: {}
        submit: {}
      profiles:
      - description: Analyze files using static analysis techniques and extract information
          from the file without executing it, such as metadata, strings, and structural
          information.
        display_name: '[OFFLINE] Static Analysis'
        name: static
        params:
          services:
            selected:
            - Filtering
            - Antivirus
            - Static Analysis
            - Extraction
            - Networking
      - description: Analyze files using static analysis techniques along with executing
          them in a controlled environment to observe their behavior and capture runtime
          activities, interactions with the system, network communications, and any malicious
          behavior exhibited by the file during execution.
        display_name: '[OFFLINE] Static + Dynamic Analysis'
        name: static_with_dynamic
        params:
          services:
            selected:
            - Filtering
            - Antivirus
            - Static Analysis
            - Extraction
            - Networking
            - Dynamic Analysis
      - description: Combine traditional static analysis techniques with internet-connected
          services to gather additional information and context about the file being analyzed.
        display_name: '[ONLINE] Static Analysis'
        name: static_with_internet
        params:
          services:
            selected:
            - Filtering
            - Antivirus
            - Static Analysis
            - Extraction
            - Networking
            - Internet Connected
      - description: Perform comprehensive file analysis using traditional static and
          dynamic analysis techniques with internet access.
        display_name: '[ONLINE] Static + Dynamic Analysis'
        name: static_and_dynamic_with_internet
        params:
          service_spec:
            CAPE:
              routing: internet
            URLDownloader:
              proxy: localhost_proxy
          services:
            selected:
            - Filtering
            - Antivirus
            - Static Analysis
            - Extraction
            - Networking
            - Internet Connected
            - Dynamic Analysis
      sha256_sources: []
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
      temporary_keys: {}
      verdicts:
        highly_suspicious: 700
        info: 0
        malicious: 1000
        suspicious: 300
    system:
      constants: assemblyline.common.constants
      organisation: ACME
      type: production
    ui:
      ai_backends:
        api_connections:
        - api_type: openai
          chat_url: https://api.openai.com/v1/chat/completions
          headers:
            Content-Type: application/json
          model_name: gpt-3.5-turbo
          proxies: null
          verify: true
        - api_type: openai
          chat_url: https://api.openai.com/v1/chat/completions
          headers:
            Content-Type: application/json
          model_name: gpt-4
          proxies: null
          verify: true
        enabled: false
        function_params:
          assistant:
            max_tokens: 1024
            options:
              frequency_penalty: 0
              presence_penalty: 0
              temperature: 0
              top_p: 1
            system_message: '## Context

              You are the Assemblyline (AL) AI Assistant. You help people answer their
              questions and other requests interactively

              regarding Assemblyline. $(EXTRA_CONTEXT)


              ## Style Guide


              - Your output must be formatted in standard Markdown syntax

              - Highlight important information using backticks

              - Your answer must be written in plain $(LANG).

              '
          code:
            max_tokens: 1024
            options:
              frequency_penalty: 0
              presence_penalty: 0
              temperature: 0
              top_p: 1
            system_message: '## Context


              You are an assistant that provides explanation of code snippets found in
              AssemblyLine (AL),

              a malware detection and analysis tool. $(EXTRA_CONTEXT)


              ## Style Guide


              - Your output must be formatted in standard Markdown syntax

              - Highlight important information using backticks

              - Your answer must be written in plain $(LANG).

              '
            task: 'Take the code file below and give me a two part result:


              - The first part is a short summary of the intent behind the code titled
              "## Summary"

              - The second part is a detailed explanation of what the code is doing titled
              "## Detailed Analysis"

              '
          detailed_report:
            max_tokens: 2048
            options:
              frequency_penalty: 0
              presence_penalty: 0
              temperature: 0
              top_p: 1
            system_message: '## Context


              You are an assistant that summarizes the output of AssemblyLine (AL), a
              malware detection and analysis tool.

              Your role is to extract information of importance and discard what is not.
              $(EXTRA_CONTEXT)


              ## Style Guide


              - Your output must be formatted in standard Markdown syntax

              - Highlight important information using backticks

              - Your answer must be written in plain $(LANG).

              '
            task: "Take the Assemblyline report below in yaml format and create a two\
              \ part result:\n\n- The first part is a one or two paragraph executive summary\
              \ titled \"## Executive Summary\" which\n  provides some high level highlights\
              \ of the results\n- The second part is a detailed description of the observations\
              \ found in the report, this section\n  is titled \"## Detailed Analysis\"\
              \n"
          executive_summary:
            max_tokens: 1024
            options:
              frequency_penalty: 0
              presence_penalty: 0
              temperature: 0
              top_p: 1
            system_message: '## Context


              You are an assistant that summarizes the output of AssemblyLine (AL), a
              malware detection and analysis tool. Your role

              is to extract information of importance and discard what is not. $(EXTRA_CONTEXT)


              ## Style Guide


              - Your output must be formatted in standard Markdown syntax

              - Highlight important information using backticks

              - Your answer must be written in plain $(LANG).

              '
            task: 'Take the Assemblyline report below in yaml format and summarize the
              information found in the

              report into a one or two paragraph executive summary. DO NOT write any headers
              in your output.

              '
      alerting_meta:
        important:
        - original_source
        - protocol
        - subject
        - submitted_url
        - source_url
        - url
        - web_url
        - from
        - to
        - cc
        - bcc
        - ip_src
        - ip_dst
        - source
        subject:
        - subject
        url:
        - submitted_url
        - source_url
        - url
        - web_url
      allow_malicious_hinting: false
      allow_raw_downloads: true
      allow_replay: false
      allow_url_submissions: true
      allow_zip_downloads: true
      api_proxies: {}
      audit: true
      audit_login: false
      banner: null
      banner_level: info
      community_feed: https://alpytest.blob.core.windows.net/pytest/community.json
      debug: false
      default_quotas:
        concurrent_api_calls: 10
        concurrent_async_submissions: 0
        concurrent_submissions: 5
        daily_api_calls: 0
        daily_submissions: 0
      default_zip_password: infected
      discover_url: null
      download_encoding: cart
      email: null
      enforce_quota: true
      external_links: []
      external_sources: []
      fqdn: localhost
      ingest_max_priority: 250
      read_only: false
      read_only_offset: ''
      rss_feeds:
      - https://alpytest.blob.core.windows.net/pytest/stable.json
      - https://alpytest.blob.core.windows.net/pytest/services.json
      - https://alpytest.blob.core.windows.net/pytest/community.json
      - https://alpytest.blob.core.windows.net/pytest/blog.json
      secret_key: This is the default flask secret key... you should change this!
      services_feed: https://alpytest.blob.core.windows.net/pytest/services.json
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
      url_submission_auto_service_selection:
      - URLDownloader
      url_submission_headers:
        User-Agent: Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko)
          Chrome/110.0.0.0 Safari/537.36
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

    * [Cluster deployment update](../../cluster/general#update-your-deployment)
    * [Appliance deployment update](../../appliance/kubernetes-microk8s#updating-the-current-deployment)

## Exhaustive configuration file documentation

All parameters of each configuration section will be thoroughly documented in their respective pages.

Here are the links to the different section documentations:

* [Authentication (auth:)](../authentication)
* [Core components  (core:)](../core)
* [Data sources (datasources:)](../datasources)
* [Database (datastore:)](../datastore)
* [File storage (filestore:)](../filestore)
* [Logging (logging:)](../logging)
* [Retrohunt (retrohunt:)](../retrohunt)
* [Services (services:)](../services)
* [Submission (submission:)](../submission)
* [System (system:)](../system)
* [User Interface (ui:)](../ui)
