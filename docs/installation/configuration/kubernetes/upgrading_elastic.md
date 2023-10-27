As of version 4.3.0 of the Assemblyline helm chart, we'll start using Elasticsearch 8 for the datastore and log-storage, if enabled.

To migrate your system to the newest version of the chart and Elasticsearch, take note of the following before performing the upgrade:

### Elasticsearch (datastore/log-storage)
If you're using an exact copy-and-paste of the `values.yaml` with some modifications, note the following (`datastore` will be used as an example):

!!! note "Upgrading `log-storage` configuration for internal TLS/SSL involves different environment variables (`DATASTORE_*` → `LOGGING_*`)"

```yaml
datastore:
  imageTag: "8.10.2" # Tag statically set to the latest version of ES at the time of writing
  createCert: False # Prevent the chart from creating it's own certificates
  roles: # Assign the specific roles required for database
    - master
    - data
    - data_content
    - data_hot
    - data_warm
    - data_cold
    # Only if using Elasticsearch instance for logging
    # - ingest
  secret:
    enabled: false  # Disable auto-password generation
  esConfig:
    elasticsearch.yml: |
      ...
      # Suppress warning logs from certain loggers
      logger.org.elasticsearch.cluster.coordination.ClusterBootstrapService: ERROR
      logger.com.amazonaws.auth.profile.internal.BasicProfileConfigFileLoader: ERROR
      logger.org.elasticsearch.cluster.coordination.Coordinator: ERROR
      logger.org.elasticsearch.deprecation: ERROR
      logger.org.elasticsearch.xpack.ml.inference.loadingservice.ModelLoadingService: ERROR

      # Don't create a deprecation index
      cluster.deprecation_indexing.enabled: false

      # Below configurations are used when `enableInternalEncryption: true`
      # xpack.security.http.ssl.enabled: ${DATASTORE_SSL_ENABLED}
      # xpack.security.http.ssl.verification_mode: full
      # xpack.security.http.ssl.key: ${DATASTORE_SSL_KEY}
      # xpack.security.http.ssl.certificate: ${DATASTORE_SSL_CERTIFICIATE}
```


### Kibana
As of Elastic v8, Kibana isn't allowed to use the local superuser to authenticate with the cluster. As a result, we've created a job that will generate a service account token that Kibana will use instead.

This will involve creating a new secret in the Assemblyline namespace:
```yaml
# Initalizes secret with a temporary value, will be replaced by job upon helm (install|upgrade)
apiVersion: v1
kind: Secret
metadata:
    name: kibana-service-token
stringData:
    token: ""
```

Using the following command:
```bash
kubectl create secret generic kibana-service-token --from-literal=token="" -n <al_ns>
```

As well as the following changes to the Kibana chart values:
```yaml
kibana:
...
extraEnvs:
  - name: ELASTICSEARCH_SERVICEACCOUNTTOKEN
    valueFrom:
      secretKeyRef:
        name: kibana-service-token
        key: token
kibanaConfig:
  kibana.yml: |
    elasticsearch:
      hosts: ['${ELASTICSEARCH_HOSTS}']
      serviceAccountToken: '${ELASTICSEARCH_SERVICEACCOUNTTOKEN}'
      # ssl.certificate: '${ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES}'
```

No changes should be required for Beats, Logstash, and APM server

### Upgrade Checklist
- [x] (Optional) Pause system processing using the toggles on the Ingestion & Dispatch cards in the Dashboard view
- [x] Update Elasticsearch configuration(s) in `values.yaml`
- [x] Create a new Secret for Kibana's service token with `""` value
- [x] Update Kibana configuration in `values.yaml`

If all these items are checked off, you can proceed with a `helm upgrade` of your deployment.
