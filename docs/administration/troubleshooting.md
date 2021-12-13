# Troubleshooting / FAQ

We will update this page with typical issues and solutions.

You can post your question to our [Assemblyline Google Group](https://groups.google.com/g/cse-cst-assemblyline) or join our [Discord](https://discord.gg/GUAy9wErNu) community!


=== "Core"
    === "Statistics"
        ??? question "Why is the hits counter for a certain signature not incrementing even though it hit 5000 times in the last hour?"

            Rules' hit count are not calculated live. There is an external process that calculates them daily for performance optimization.

            However, you can click on the rule itself and it does calculate the stats for that specific rule and updates it right away.
    === "Updater"
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
=== "Kubernetes"
    ??? question "Connection timeouts to external domains"
        By default, coreDNS is configured to the Google's Public DNS when trying to resolve external domains outside the cluster.

        You can configure it to use a different DNS via:
        ```sudo microk8s disable dns && sudo microk8s enabled dns:1.1.1.1```

        If this still poses an issue, refer to: [Teleport - Troubleshooting Kubernetes Networking Issues](https://goteleport.com/blog/troubleshooting-kubernetes-networking/)
    ??? question "How can I monitor the status of the deployment/cluster?"
        The quickest way to monitor the status of your cluster is:
        ```kubectl get pods -n <assemblyline_namespace>```

        There are other tools such as [k9s](https://k9scli.io/) and [Lens](https://k8slens.dev/) that allow you to monitor your cluster in a more user-friendly manner.
    ??? question "Are HPAs adjustable?"
        Depending on the amount of activity you're receiving, you'll likely have to tweak the *TargetUsage and *ReqRam settings in your values.yaml for your particular deployment, either to cause it to scale faster or slower.

        Refer to: [Kubernetes - Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) for more information.
    ??? question "NGINX 504 but everything seems to be running"
        It's possible the domain you're accessing the UI with doesn't match the setting ```configuration.ui.fqdn``` in your values.yaml.
        If your setting is set to 'localhost' but you're accessing the UI using '192.0.0.1.nip.io', there is no ingress path using '192.0.0.1.nip.io' as a base.

        The simplest solution is to update your values.yaml to the appropriate FQDN and redeploy.
    ??? question "Is it possible to mount an internal root CA bundles into core components to use?"
        Yes! This would involve creating a configmap containing your CA bundle and using ```coreVolumes, coreMounts, and coreEnv``` in your values.yaml to pass that information onto the core deployments.

        An example of this configuration would be:
        ```yaml
        coreEnv:
          - name: REQUESTS_CA_BUNDLE
            value: /usr/share/internal-ca
        coreMounts:
          - name: internal-certs
            mountPath: /usr/share/internal-ca
        coreVolumes:
          - name: internal-certs
            configMap:
              name: internal-certs-config
        ```


=== "Services"
    === "General"
        ??? question "TASK PRE-EMPTED"
            A service task can pre-empt for a number of reasons, such as:

            1. Container being killed due to OOM (Out of Memory)
                * Increase service memory limits
                * Debug service memory usage with sample
            2. Service instance ran beyond the service timeout
                * Increase service timeout
                * Debug service runtime with sample
            3. The service result didn't make it back to service-server
                * Check the state of service-server deployments
                * Inspect network policies

        ??? question "Is there a limit resources to allocate to a service instance?"

            No! The sky's the limit (or more accurately), you're bound to the resources allocated on your cluster.
            That being said, it's best practice to use what you need rather than what you want (especially if it's unwarranted).

        ??? question "Can I deploy a 4.X service on a 4.Y Assemblyline system (where X,Y are system versions: X < Y)"

            Yes and no. You can add a service with to the system with a lesser system version but it won't be enabled due to potential compatibility issues.
            It's advised to rebuild the service and tag with the system version that you want to deploy on.

    === "Service Updater"
        ??? question "My service doesn't seem to be getting any signatures for analysis.."
            Check the following on your service's updater instance:

            1. Does your service have the following environment variables set: updates_host, updates_port, updates_key
                * Edit the deployment manually
                * Disable the service, delete the related deployments from Kubernetes, restart scaler, then re-enable service
            2. Is there any downloaded signatures in ```/tmp/updater/update_dir*/*/``` of the updater container?
                * If empty, it suggests there was a problem pulling the signatures from Assemblyline
                * If some sources are missing, it suggests there was a problem pulling signatures from that source

        ??? question "I've added my signature sources, but nothing shows up when searching the Signature index.."

            1. Are the source links accessible from the cluster?
            2. Is the authentication for each source valid?
            3. How frequently is the service updater configured to check for source updates?

        ??? question "If I modify signatures using the Assemblyline client or the API, will the service get these rules?"

            Yes! Changes made involving our API trigger messaging events made to other components to Assemblyline causing the system to be more responsive.
            In the case of services with updaters, they'll be notified immediately and the service will gather the new rules after a submissions has processed.
