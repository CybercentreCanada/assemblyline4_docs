# Core

This section contains troubleshooting steps for core components of Assemblyline

## API / UI

??? question "When deployed behind a proxy, I just get a login loop upon sign in"
    By default, Assemblyline will try to perform validation of the session IP and user-agent. Since operating behind a proxy could cause either or both to fail validation, you can disable these checks with the following configuration(s):
    ```yaml
    ui:
      validate_session_ip: false
      validate_session_useragent: false
    ```

## Statistics

??? question "Why is the hits counter for a certain signature not incrementing even though it hit 5000 times in the last hour?"

    Rules' hit count are not calculated live. There is an external process that calculates them daily for performance optimization.

    However, you can click on the rule itself and it does calculate the stats for that specific rule and updates it right away.

## Updater

??? question "Failed to establish a new connection: [Errno 110] Connection timed out"
            Depending on the host mentioned in the error, ensure the deployment has access to the registry and its able to call the associated API.

            The following modifications will have to be made to your values.yaml:
            === "External Registry"
                Let's say the domain of the registry is 'registry.local' and is hosted on port 443
                ```yaml
                configuration:
                services:
                    image_variables:
                        REGISTRY: "registry.local/"
                    allow_insecure_registry: true
                ```
            === "Local Registry"
                Let's say the local registry is hosted on port 32000
                ```yaml
                configuration:
                services:
                    image_variables:
                        REGISTRY: "localhost:32000/"
                    update_image_variables:
                        REGISTRY: "<HOST_IP>:32000/"
                    allow_insecure_registry: true
                ```
