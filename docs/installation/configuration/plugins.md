# UI plugin section

The plugin configuration section (`uiPlugins:`) of the `values.yaml` allows you to enable and configure both the built-in and your own custom plugins.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "UI plugin section configuration example"
    ```yaml
    ...
    uiPlugins: # configure/setup external lookup type plugins.
      lookup:
        # Enable lookup plugins
        # These are disabed by default. Set to `true` to enable them (note: individual plugins must still be enabled).
        enabled: false

        # Mapping of plugins to setup.
        # You can add your own custom plugins to this mapping as long as they implement the correct interface.
        plugins:

          <lookupName>: # The name of the lookup plugin (eg. virustotal). This will be used in the deployment definitions.

            # Enable or disable this particular plugin.
            # Note: to enable, the upper level `uiPlugins.lookup.enabled` must also be `true`.
            enabled: false

            # The registry and image name of the plugin image.
            image: cccs/assemblyline-ui-plugin-lookup-virustotal

            # (optional) The image tag to use. If left blank it will default to the current Assemblyline version.
            imageTag:

            # The number of instances to run.
            instances: 1

            # CPU and RAM requests and limits.
            reqCPU: 250m
            reqRam: 256Mi
            limRam: 256Mi

            # Port the plugin container listens on.
            containerPort: 8000

            # (optional) Supply the name of an existing configMap that contains configuration data for the plugin.
            # Use this OR the below `configMapData`.
            configMapName:

            # (optional) Configuration data for the plugin to be exposed as environment variables.
            configMapData: |-
              VT_VERIFY: true
              MAX_TIMEOUT: "3"
              API_URL: "https://www.virustotal.com/api/v3"
              FRONTEND_URL: "https://www.virustotal.com/gui/search"
              CLASSIFICATION: null

            # (optional) Secret configuration data for the plugin to be exposed as environment variables.
            # Use this OR the one of the below `secretName`, `secretKeys`.
            secretData: |-
              VT_API_KEY: ""

            # (optional) Supply the name of an existing secret that contains configuration data for the plugin.
            secretName:

            # (optional) Individually use keys from existing secrets
            # supplied as a list, eg:
            # - varName: VT_API_KEY
            #   secretName: ui-plugin-lookup-virustotal-apikey
            #   secretKey: API_KEY
            secretKeys: []

            # (optional) Security context for the plugin to run under.
            securityContext:
              user: 1000
              group: 1000
              fsGroup: 1000
          ...
    ...

    # add the configured plugins to the AL UI config section.
    configuration:
      ...
      ui:
        external_sources:

        # Display name of the lookup source.
        - name: virustotal

          # Full url to the plugin microservice api.
          # (this will be the lowercase `lookupName` key with a `ui-plugin-lookup-` prefix)
          url: http://ui-plugin-lookup-virustotal:8000

          # (optional) Minimum classification require to access the upstream service.
          classification:

          # (optional) Maximum classification that can be sumbitted to the upstream service.
          max_classification:
          ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
