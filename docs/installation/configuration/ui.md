# User Interface section

The user interface configuration section (`ui:`) of the configuration file allows you to modify the parameters of the user interface and API Server.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "User interface section configuration example"
    ```yaml
    ...
    ui:
      # Show the malicious hinting checkbox when submitting files
      allow_malicious_hinting: false

      # Allow malicious files to be download in raw format
      allow_raw_downloads: true

      # Allow URL submissions to be processed in the system
      allow_url_submissions: true

      # Audit API queries
      audit: true

      # String to be displayed in the banner
      banner: null

      # Color of the banner (info, success, error, warning)
      banner_level: info

      # Turn on/off  debug mode
      debug: false

      # Default encoding for downloaded files
      download_encoding: cart

      # Email address users can reach the admins at
      email: null

      # Enforce API and submissions quotas or not
      enforce_quota: true

      # Domain for your deployment (Especially important for kubernetes deployments)
      fqdn: localhost

      # Maximum submission priority for ingestion tasks
      ingest_max_priority: 250

      # Make the UI read only
      # (Not supported in the new UI yet)
      read_only: false

      # Time offset for queries done in raed only mode
      # (Not supported in the new UI yet)
      read_only_offset: ''

      # Secret key for your flask app (API)
      # You should definitely change this!
      secret_key: This is the default flask secret key... you should change this!

      # Timeout after which a stale session is no longer valid
      session_duration: 3600

      # Fields to generate statistics on
      statistics:
        # Statistics in the alert view
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

        # Statistics in the submission view
        submission:
        - params.submitter

      # Terms of service for the deployment (in markdown format)
      tos: null

      # Lockout the user after they agree to the terms of service
      # (requires an admin to enable their account)
      tos_lockout: false

      # List of email addresses to notify when a user agreed to the TOS
      # and its account is locked out
      tos_lockout_notify: null

      # Headers added to fetch the files during URL submissions
      url_submission_headers: {}

      # Proxy configuration to use while fetching the file during URL submissions
      url_submission_proxies: {}

      # Should we validate that the session comes from the same IP?
      validate_session_ip: true

      # Should we validate that the session uses the same user agent
      validate_session_useragent: true
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
