# Authentication section

Assemblyline comes with a built-in user management database, so no external identity sources are required.
However, to facilitate user management in larger organizations you can integrate Assemblyline with external identity providers.

The authentication section (`auth:`) of the configuration files contains all the different parameters that you can change to turn on/off the different authentication features that Assemblyline supports.

??? info "Default values for the authentication section"
    ```yaml
    ...
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
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.

## Parameter definitions

The `auth` configuration block has a few parameters at the top level that help you turn on or off a few security features supported in the system.

Here is an example of a configuration block with those top-level parameters and an explanation of what they do:

???+ example "Top-level parameters"
    ```yaml
    auth:
        # Turns on/off two-factor authentication in the system
        allow_2fa: true

        # Turn on/off usage of API Keys in the system
        #  NOTE: if you turn this off, this will severely limit API access
        allow_apikeys: true

        # Turn on/off usage of extended API via the API keys
        allow_extended_apikeys: true

        # Turn on/off usage of security token as two-factor authentication (ex: yubikeys)
        allow_security_tokens: true
    ```

### Internal authenticator

The configuration block at `auth.internal` allows you to configure the Assemblyline internal authenticator.

Here is an example of a configuration block with inline comments about the purpose of every single parameter:

???+ example "Internal auth configuration example"
    ```yaml
    auth:
        internal:
            # Enable or disable the internal authenticator
            enabled: true

            # Time in seconds the user will have to wait after
            # too many authentication failures
            failure_ttl: 60

            # Number of authentication failures before temporarily
            # locking down the user
            max_failures: 5

            # Password complexity requirements for the system
            password_requirements:
            # Are lowercase characters mandatory?
            lower: false

            # What is the minimal password length
            min_length: 12

            # Are numbers mandatory?
            number: false

            # Are special characters mandatory?
            special: false

            # Are uppercase characters mandatory?
            upper: false

            signup:
            # Can a user automatically signup for the system
            enabled: false

            # Configuration block for GC Notify signup and password reset
            # see: https://notification.canada.ca/
            notify:
                activated_template: null
                api_key: null
                authorization_template: null
                base_url: null
                password_reset_template: null
                registration_template: null

            # Configuration block for SMTP signup and password reset
            smtp:
                # Email address used for sender
                from_adr: null

                # Host of the SMTP server
                host: null

                # Password for the SMTP server
                password: null

                # Port of the SMTP server
                port: 587

                # Should we communicate with SMTP server via TLS?
                tls: true

                # User to authenticate to the SMTP server
                user: null

            # Email patterns that will be allowed to
            #  automatically signup for an account
            valid_email_patterns:
            - .*
            - .*@localhost
    ```

### LDAP Authentication

The configuration block at `auth.ldap` allows you to easily add authentication via your LDAP server. The LDAP authentication module will be able to automatically assign roles, classification, avatar, name, and email address based on the properties of the LDAP user and the groups it is a member of.

Here is an example configuration block to add to your configuration file that will allow you to connect to the docker-test-openldap server from: [https://github.com/rroemhild/docker-test-openldap](https://github.com/rroemhild/docker-test-openldap)

???+ example "LDAP configuration example"
    ```yaml
    auth:
        internal:
            # Disable internal login, you could also leave it on if you want
            enabled: false
        ldap:
            # Should LDAP be enabled or not?
            enabled: true

            # DN of the group or the user who will get admin privileges
            admin_dn: cn=admin_staff,ou=people,dc=planetexpress,dc=com

            # Auto-create users if they are missing, this means
            #  that if a user exists in LDAP, Assemblyline will create an
            #  account for it upon the first login
            auto_create: true

            # Should we automatically sync roles, classification, avatar
            #  email, name... with the LDAP server upon each login?
            auto_sync: true

            # Base DN for the users
            base: ou=people,dc=planetexpress,dc=com

            # Password used to query the LDAP server
            bind_pass: null

            # User use to query the LDAP server
            bind_user: null

            classification_mappings: {}

            # Name of the field containing the email address
            email_field: mail

            # How the group lookup is queried
            group_lookup_query: (&(objectClass=Group)(member=%s))

            # Name of the field containing the user's avatar
            image_field: jpegPhoto

            # Type of image used to store the avatar
            image_format: jpeg

            # Name of the field containing the user's name
            name_field: cn

            # DN of the group or the user who will get signature_importer role
            signature_importer_dn: null

            # DN of the group or the user who will get signature_manager role
            signature_manager_dn: null

            # Field name for the UID
            uid_field: uid

            # URI to the LDAP server
            uri: ldaps://<ldap_ip_or_domain>:636
    ```

### OAuth Authentication

The configuration block at `auth.oauth` allows you to add OAuth authentication to your system. Assemblyline OAuth module is configurable enough to allow you to use almost any OAuth provider.

It has been thoroughly tested with:

* [Microsoft Accounts](https://account.microsoft.com/account)
* [Google Accounts](https://www.google.com/account/about/)
* [Auth0](https://auth0.com/)
* [Microsoft Azure Active Directory Accounts](https://docs.microsoft.com/azure/active-directory/)

Here is an exhaustive configuration block that explains every single parameter from the OAuth configuration block:

???+ example "Exhaustive OAuth configuration example"
    ```yaml
    auth:
        internal:
            # Disable internal login, you could also leave it on if you want
            enabled: false
        oauth:
            # Should OAuth authentication be enabled or not
            enabled: true

            # Should we try to pull the user's avatar using gravatar
            gravatar_enabled: false

            # OAuth providers configuration block, you can have as many OAuth
            #  providers as you want
            providers:
                # Name of the provider displayed in the UI
                local_provider:
                    # Auto-create users if they are missing, this means
                    #  that if a user exists in the OAuth provider, Assemblyline
                    #  will create an account for it upon the first login
                    #     WARNING: If you set it to true for let's say Google's
                    #              OAuth provider, anyone with a google account
                    #              essentially has access to your system
                    auto_create: true

                    # Should we automatically sync roles, classification, avatar
                    #  email, name... with the OAuth provider upon each login?
                    auto_sync: true

                    # Automatic role and classification assignments
                    auto_properties:
                        # any user with a @localhost.local email will be given
                        #  TLP:Amber classification
                        - field: email
                          pattern: .*@localhost\.local$
                          type: classification
                          value: "TLP:A"
                        # any user within the admins-sg will be made
                        #  administrator in the system
                        - field: groups
                          pattern: ^admins-sg$
                          type: role
                          value: admin

                    # URL used to get the access token
                    access_token_url: https://oauth2.localhost/token

                    # Base URL for downloading the user's and groups info
                    api_base_url: https://openidconnect.localhost/

                    # URL used to authorize access to a resource
                    authorize_url: https://localhost/oauth2/auth

                    # ID of your application to authenticate to the OAuth
                    #  provider
                    client_id: null

                    # Password to your application to authenticate to the
                    #  OAuth provider
                    client_secret: null

                    # Keyword arguments passed to the different URLs
                    #  (to set the scope for example)
                    client_kwargs:
                        scope: openid email profile

                    # URL used to verify if a returned JWKS token is valid
                    jwks_uri: https://localhost/oauth2/certs

                    # Name of the field that will contain the user ID
                    uid_field: uid

                    # Should we generate a random username for the
                    #  authenticated user?
                    uid_randomize: false

                    # How many digits should we add at the end of the username?
                    uid_randomize_digits: 0

                    # What is the delimiter used by the random name generator?
                    uid_randomize_delimiter: "-"

                    # Reged used to parse and email address and capture parts
                    #  to create a user ID out of it
                    uid_regex: ^(.*)@(\w*).*$

                    # Format of the user ID based on the captured parts from the regex
                    uid_format: '{}-{}'

                    # Should we use the new callback method?
                    use_new_callback_format: true

                    # Path from the base_url to fetch the user info
                    user_get: user/info

                    # Path from the base to fetch the group info
                    user_groups: group/info

                    # Field return by the group info API call that contains the
                    #  list of groups
                    user_groups_data_field: null

                    # Name of the field in the list of groups that contains the
                    #  name of the group
                    user_groups_name_field: null
    ```


Here is an example configuration block that would let you use Auth0 if you would change your `client_id` and `client_secret` and that you would change the `tenant_name` to yours:

???+ example "Auth0 configuration example"
    ```yaml
    auth:
        internal:
            # Disable internal login, you could also leave it on if you want
            enabled: false
        oauth:
            # Enable oAuth
            enabled: true

            # Setup the auto0 provider
            providers:
                auth0:
                    # It is safe to auto-create users here
                    # because it is your OAuth tenant
                    auto_create: true
                    auto_sync: true

                    # Put your client ID and secret here
                    client_id: <YOUR_CLIENT_ID>
                    client_secret: <YOUR_CLIENT_SECRET>

                    client_kwargs:
                        scope: openid email profile

                    # Set your tenant's name in the following URLs
                    access_token_url: https://<TENANT_NAME>.auth0.com/oauth/token
                    api_base_url: https://<TENANT_NAME>.auth0.com/
                    authorize_url: https://<TENANT_NAME>.auth0.com/authorize
                    jwks_uri: https://<TENANT_NAME>.auth0.com/.well-known/jwks.json

                    user_get: userinfo
    ```
