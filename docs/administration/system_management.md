# System Management

## Assemblyline Major Upgrades

### Pause Processing

If upgrading the framework version (4.X → 4.Y, where X < Y) starting from [Release 4.2.0.121](https://github.com/CybercentreCanada/assemblyline/releases/tag/v4.2.0.stable121)+ or if performing another major change that affects Assemblyline, it's strongly recommended to pause the Ingester and Dispatcher and allow services to complete what's currently been tasked to them. This can be done by:

- Changing the state(s) using the `/api/v4/system/status/<component>/` API and setting the variable `active=false`
- Using the toggle found in the Dashboard and switching Dispatcher/Ingester to the disabled state.

This ensures that if there are any breaking changes between the core and services during a major upgrade (ie. framework), the services will complete their tasks with what they were compatible with.

This also gives an extended period to upgrade services to the compatible framework version before resuming processing.

!!! danger
    Systems upgrading before [Release 4.2.0.121](https://github.com/CybercentreCanada/assemblyline/releases/tag/v4.2.0.stable121) are strongly advised to upgrade to that release at a minimum and pause the Ingester/Dispatcher before proceeding to upgrade to later releases. Alternatively, you can stop submitting files to your system and let existing submissions complete and then perform the upgrade.

### Upgrade Deployment (Core)

At this stage, you're ready to upgrade the core system and you shouldn't have any services actively processing.

Depending on the type of deployment (Kubernetes/Docker), the method of upgrading differs:

-   Docker: Update `SERVICE_VERSION` in .env file if necessary, then:

    ```bash
    docker-compose pull && docker-compose up -d
    ```

-   Kubernetes: Update `release` in values.yaml if necessary, then:

    ```bash
    helm upgrade <deployment_name> /path/to/assemblyline-helm-chart/assemblyline -f /path/to/al_deployment/values.yaml -n <deployment_namespace>
    ```

### (Optional) Upgrade Services to Compatible Framework

At this stage, your deployment has been successfully upgraded and your core components should be upgraded to a newer framework release.

While the system is still paused, on the Services page you should find that all your services are in a disabled state as they're assumed to be incompatible with the rest of the system. If using public services, there should be advertisements to upgrade to the latest compatible version.

If an advertisement doesn't appear for a service, it implies that there is no release tagged with a compatible version and requires a rebuild of the service using the more recent stable/dev tag.

Once all services are upgraded, you can resume processing by toggling the Dispatcher/Ingester back to an active status or changing the state using the API and setting `active=true`

## Assemblyline Minor Upgrades (4.X → 4.X)

This implies the changes should have little to no impact on systems within the same framework version.

Setting the system to a paused state before upgrading isn't required.

## Single Node Elasticsearch Maintenance

When running Elasticsearch in a single node deployment, it will sometimes drop to a `yellow` status without a real issue.
This typically occurs when an index has (because of templating or defaults), its `number_of_replicas` set to more than zero.
Since the single node deployment can't create a redundant replica, this forces the node to a yellow status.
The details on the settings around this can be a little different depending on which version of Elasticsearch you are using, but the following should be true for versions 7 and 8.

To check if this is the case for your Elasticsearch node, running the following command will reset the replicas to zero for all indices:

```bash
curl -XPUT -k -u "${ELASTIC_USERNAME}:${ELASTIC_PASSWORD}" https://localhost:9200/_all/_settings -H 'Content-Type: application/json' -d '{"index" : {"number_of_replicas" : 0}}'
```

You may need to replace `https` with `http` depending on your other settings. If your Elasticsearch doesn't return to a green status within a few minutes of running the command you may have other issues and should look for support.

If the issue reoccurs often enough to be an issue for you, it should be preventable by ensuring no templates have a non-zero replica value set and creating an index template like the following to establish zero as the default value.

```json
{
   "index_patterns" : ["*"],
   "order" : 0
   "settings" : {
       "number_of_replicas" : "0"
   }
}
```
