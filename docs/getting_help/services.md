# Services

This section contains troubleshooting steps for service components of Assemblyline

## General

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

??? question "Every time I update my service, the values/parameters are getting reset.."

    This can stem from loading the service initially with production values which causes an issue with the service's service_delta.
    It's best practice to add a service using the default manifest and update the values after it's been registered.

    If you have many services that require an update in values or if there's many values to update in a single service, consider using the Assemblyline client.

## Service Updater

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
