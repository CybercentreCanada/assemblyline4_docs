# Internal Encryption (TLS/SSL)

!!! warning "If enabling this feature post-deployment, consider pausing the system before proceeding to upgrade"

As of [4.3.1.0](https://github.com/CybercentreCanada/assemblyline/releases/tag/v4.3.1.stable0), the [Assemblyline Helm Chart](https://github.com/CybercentreCanada/assemblyline-helm-chart)
supports enabling end-to-end encryption internally between components.

The flag `enableInternalEncryption` found in the values.yaml is disabled by default
(as to not conflict with pre-existing deployments) but can be enabled. Enabling the feature causes Helm to generate a
Root CA for the release (`{ .Release.Name }.internal-generated-ca`) that's only used to sign certificates for servers in Assemblyline.

## Changes for other Helm charts

Enabling this feature will require some changes to the values used by the accompanying charts that we use.

For example, to enable the feature with the default values using Helm-generated Root CA for verification:

!!! note "Generated certificates are stored in Secrets as `<server_hostname>-cert`"
    Each Secret has two keys: `tls.crt`, `tls.key` which has their respective public certificate and private key

=== "Elasticsearch"
    In this example, I'll mention the changes to the `datastore` but this will also apply to `log-storage` when `seperateInternalELKStack: true`
    === "7.17.3"
        ```yaml
        datastore:
          # Change of protocol for readiness probes
          protocol: https

          # By default, we configure Elasticsearch instances (including the log-storage) to use
          # the contents of '/usr/share/elasticsearch/config/http_ssl/' when settings up HTTPS
          # This can be overridden by setting the environment variables directly, see values.yaml for more details
          extraVolumes: |
            - name: elastic-certificates
              emptyDir: {}
            - name: datastore-master-cert
              secret:
                secretName: datastore-master-cert
          extraVolumeMounts: |
            - name: elastic-certificates
              mountPath: /usr/share/elasticsearch/config/certs
            - name: datastore-master-cert
              mountPath: /usr/share/elasticsearch/config/http_ssl
              readOnly: true
        ```
    === "8+"
        ```yaml
        datastore:
          # Change of protocol for readiness probes
          protocol: https
          esConfig:
            elasticsearch.yml: |
              ingest.geoip.downloader.enabled: false
              logger.level: WARN
              xpack.security.enabled: true
              xpack.security.transport.ssl.enabled: true
              xpack.security.transport.ssl.verification_mode: certificate
              xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
              xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
              xpack.security.http.ssl.enabled: ${DATASTORE_SSL_ENABLED}
              xpack.security.http.ssl.verification_mode: full
              xpack.security.http.ssl.key: ${DATASTORE_SSL_KEY}
              xpack.security.http.ssl.certificate: ${DATASTORE_SSL_CERTIFICIATE}

          # By default, we configure Elasticsearch instances (including the log-storage) to use
          # the contents of '/usr/share/elasticsearch/config/http_ssl/' when settings up HTTPS
          # This can be overridden by setting the environment variables directly, see values.yaml for more details
          extraVolumes: |
            - name: elastic-certificates
              emptyDir: {}
            - name: datastore-master-cert
              secret:
                secretName: datastore-master-cert
          extraVolumeMounts: |
            - name: elastic-certificates
              mountPath: /usr/share/elasticsearch/config/certs
            - name: datastore-master-cert
              mountPath: /usr/share/elasticsearch/config/http_ssl
              readOnly: true
        ```
=== "Kibana"
    === "7.17.3"
        ```yaml
        # Internally hosted Kibana instance. For more details, see: https://github.com/elastic/helm-charts/tree/7.17/kibana#configuration
        kibanaHost: https://kibana/kibana
        kibana:
          elasticsearchHosts: https://log-storage-master:9200

          # For Filebeat, Metricbeat, APM, and Kibana, we mount the Root CA in '/etc/certs/ca.crt'
          # This can be overridden by replacing the ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES environment variable
          kibanaConfig:
            kibana.yml: |
              server.ssl:
                enabled: true
                certificate: /etc/certs/kibana/tls.crt
                key: /etc/certs/kibana/tls.key
                certificateAuthorities: /etc/certs/ca.crt
              elasticsearch:
                hosts: ['${ELASTICSEARCH_HOSTS}']
                username: ${ELASTICSEARCH_USERNAME}
                password: ${ELASTICSEARCH_PASSWORD}
          extraVolumes:
            - name: root-ca
              secret:
                secretName: "<release_name>.internal-generated-ca"
            - name: kibana-cert
              secret:
                secretName: kibana-cert
          extraVolumeMounts:
            - name: root-ca
              mountPath: /etc/certs/ca.crt
              subPath: tls.crt
            - name: kibana-cert
              mountPath: /etc/certs/kibana/
              readOnly: true
        ```
    === "8+"
        ```yaml
        # Internally hosted Kibana instance. For more details, see: https://github.com/elastic/helm-charts/tree/7.17/kibana#configuration
        kibanaHost: https://kibana/kibana
        kibana:
          elasticsearchHosts: https://log-storage-master:9200

          # For Filebeat, Metricbeat, APM, and Kibana, we mount the Root CA in '/etc/certs/ca.crt'
          # This can be overridden by replacing the ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES environment variable
          extraVolumes:
            - name: init-script
              configMap:
                name: init-kibana-token
            - name: root-ca
              secret:
                secretName: "<release_name>.internal-generated-ca"
          extraVolumeMounts:
            - name: root-ca
              mountPath: /etc/certs/ca.crt
              subPath: tls.crt
          kibanaConfig:
            kibana.yml: |
              server.ssl:
                enabled: true
                certificate: /etc/certs/kibana/tls.crt
                key: /etc/certs/kibana/tls.key
                certificateAuthorities: /etc/certs/ca.crt
              elasticsearch:
                hosts: ['${ELASTICSEARCH_HOSTS}']
                serviceAccountToken: '${ELASTICSEARCH_SERVICEACCOUNTTOKEN}'
                ssl.certificateAuthorities: '${ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES:[]}'
          extraVolumes:
            - name: root-ca
              secret:
                secretName: "<release_name>.internal-generated-ca"
            - name: kibana-cert
              secret:
                secretName: kibana-cert
          extraVolumeMounts:
            - name: root-ca
              mountPath: /etc/certs/ca.crt
              subPath: tls.crt
            - name: kibana-cert
              mountPath: /etc/certs/kibana/
              readOnly: true
        ```
=== "MinIO"
    ```yaml
    # Internally hosted MinIO filestore. For more details, see: https://github.com/minio/charts/tree/main/minio#configuration
    filestore:
      tls:
        enabled: true
        certSecret: filestore-cert
        publicCrt: tls.crt
        privateKey: tls.key
    ```

## Changes to Assemblyline Configuration

Since we're enabling HTTPS on all internal endpoints, this means we also have to update the URLs that we use for connecting
to the various databases.

This includes, but is not limited to:

- `configuration.metrics.apm_server.server_url: http → https`
- `configuration.metrics.elasticsearch.hosts: http → https (for all hosts)`
- `configuration.datastore.hosts: http → https (for all hosts)`
- `configuration.filestore.*: use_ssl=False → use_ssl=True (for all hosts)`


!!! note "Filestore Configurations"

    When enabling `internalEncryption` and if using the self-generated Root CA for verification, you'll need to set the `verify` parameter to point to the location of the CA on disk (`/etc/assemblyline/ssl/al_root-ca.crt`.)

    If the CA required for verification isn't the one generated by Helm, then you'll need to mount that CA to the core containers using `coreVolumes` & `coreMounts`. You'll also need to add the mounts to `configuration.core.scaler.service_defaults.mounts` so privileged services will have access to the CA for verification.

## Changes to Ingress Controller

Because we're enabling full encryption to everything Assemblyline-related, this also includes traffic going from the ingress
controller to the appropriate services. (ie. '/' → `frontend`, '/api/' → `ui`, etc.)

A simple way of doing this is setting the `ingressAnnotation` in your values.yaml to force the ingress controller to forward the request over HTTPS rather than HTTP.

In microk8s, using their built-in ingress, you'd have to add:

`nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"`

However, this particular annotation doesn't work for all ingress controllers, so you'll need to refer to the respective documentation to find the right annotation configuration. For example, the annotation for [NGINX](https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/advanced-configuration-with-annotations/) installed via `helm` would be:

`nginx.org/ssl-services: "ui,frontend,kibana,socketio"`.

## What does this look like in practice?
For example this is what your changes would look like if:

- Enabling internal encryption (`enableInternalEncryption: true`)
- Using an internal minIO filestore that the chart deploys  (`internalFilestore: true`)
- Using the helm-generated CA for the various different servers within the cluster (`datastore` and/or `log-storage`, `kibana`, `filestore`)

!!! warning "Configuration depends on the version of the Assemblyline helm chart used in your deployment"

=== "4.2.0"
    ```diff
    - enableInternalEncryption: false
    + enableInternalEncryption: true

      internalFilestore: true

      configuration:
        core:
          ...
          metrics:
            apm_server:
    -         server_url: "http://apm:8200"
    +         server_url: "https://apm:8200"
            elasticsearch:
              hosts:
    -           ["http://${LOGGING_USERNAME}:${LOGGING_PASSWORD}@${LOGGING_HOST}"]
    +           ["https://${LOGGING_USERNAME}:${LOGGING_PASSWORD}@${LOGGING_HOST}"]
        datastore:
          ...
    -     hosts: ["http://elastic:${ELASTIC_PASSWORD}@datastore-master:9200"]
    +     hosts: ["https://elastic:${ELASTIC_PASSWORD}@datastore-master:9200"]
        filestore:
          ...
          cache:
            [
    -         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-cache&use_ssl=False",
    +         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-cache&use_ssl=True&verify=/etc/assemblyline/ssl/al_root-ca.crt",
            ]
          storage:
            [
    -         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-storage&use_ssl=False",
    +         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-storage&use_ssl=True&verify=/etc/assemblyline/ssl/al_root-ca.crt",
            ]
      ...

    # Equivalent changes for `datastore` would be made for `log-storage`
      datastore:
    +   protocol: https
        ...
        extraVolumes: |
          - name: elastic-certificates
            emptyDir: {}
    +     - name: datastore-master-cert
    +       secret:
    +         secretName: datastore-master-cert
        extraVolumeMounts: |
          - name: elastic-certificates
            mountPath: /usr/share/elasticsearch/config/certs
    +     - name: datastore-master-cert
    +       mountPath: /usr/share/elasticsearch/config/http_ssl
    +       readOnly: true
      ...

    - kibanaHost: http://kibana/kibana
    + kibanaHost: https://kibana/kibana
      kibana:
    -   elasticsearchHosts: http://log-storage-master:9200
    +   elasticsearchHosts: https://log-storage-master:9200
        ...
    +   kibanaConfig:
    +     kibana.yml: |
    +       server.ssl:
    +         enabled: true
    +         certificate: /etc/certs/kibana/tls.crt
    +         key: /etc/certs/kibana/tls.key
    +         certificateAuthorities: /etc/certs/ca.crt
    +       elasticsearch:
    +         hosts: ['${ELASTICSEARCH_HOSTS}']
    +         username: ${ELASTICSEARCH_USERNAME}
    +         password: ${ELASTICSEARCH_PASSWORD}
    +   extraVolumes:
    +     - name: root-ca
    +       secret:
    +         secretName: "<release_name>.internal-generated-ca"
    +     - name: kibana-cert
    +       secret:
    +         secretName: kibana-cert
    +   extraVolumeMounts:
    +     - name: root-ca
    +       mountPath: /etc/certs/ca.crt
    +       subPath: tls.crt
    +     - name: kibana-cert
    +       mountPath: /etc/certs/kibana/
    +       readOnly: true
      ...

      filestore:
        ...
    +   tls:
    +     enabled: true
    +     certSecret: filestore-cert
    +     publicCrt: tls.crt
    +     privateKey: tls.key
    ```
=== "4.3.0"
    ```diff
    - enableInternalEncryption: false
    + enableInternalEncryption: true

      internalFilestore: true

      configuration:
        core:
          ...
          metrics:
            apm_server:
    -         server_url: "http://apm:8200"
    +         server_url: "https://apm:8200"
            elasticsearch:
              hosts:
    -           ["http://${LOGGING_USERNAME}:${LOGGING_PASSWORD}@${LOGGING_HOST}"]
    +           ["https://${LOGGING_USERNAME}:${LOGGING_PASSWORD}@${LOGGING_HOST}"]
        datastore:
          ...
    -     hosts: ["http://elastic:${ELASTIC_PASSWORD}@datastore-master:9200"]
    +     hosts: ["https://elastic:${ELASTIC_PASSWORD}@datastore-master:9200"]
        filestore:
          ...
          cache:
            [
    -         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-cache&use_ssl=False",
    +         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-cache&use_ssl=True&verify=/etc/assemblyline/ssl/al_root-ca.crt",
            ]
          storage:
            [
    -         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-storage&use_ssl=False",
    +         "s3://${INTERNAL_FILESTORE_ACCESS}:${INTERNAL_FILESTORE_KEY}@filestore:9000?s3_bucket=al-storage&use_ssl=True&verify=/etc/assemblyline/ssl/al_root-ca.crt",
            ]
      ...

    # Equivalent changes for `datastore` would be made for `log-storage`
      datastore:
    +   protocol: https
    +   esConfig:
    +     elasticsearch.yml: |
    +       ingest.geoip.downloader.enabled: false
    +       logger.level: WARN
    +       xpack.security.enabled: true
    +       xpack.security.transport.ssl.enabled: true
    +       xpack.security.transport.ssl.verification_mode: certificate
    +       xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    +       xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/config/certs/elastic-certificates.p12
    +       xpack.security.http.ssl.enabled: ${DATASTORE_SSL_ENABLED}
    +       xpack.security.http.ssl.verification_mode: full
    +       xpack.security.http.ssl.key: ${DATASTORE_SSL_KEY}
    +       xpack.security.http.ssl.certificate: ${DATASTORE_SSL_CERTIFICIATE}
        ...
        extraVolumes: |
          - name: elastic-certificates
            emptyDir: {}
    +     - name: datastore-master-cert
    +       secret:
    +         secretName: datastore-master-cert
        extraVolumeMounts: |
          - name: elastic-certificates
            mountPath: /usr/share/elasticsearch/config/certs
    +     - name: datastore-master-cert
    +       mountPath: /usr/share/elasticsearch/config/http_ssl
    +       readOnly: true
      ...

    - kibanaHost: http://kibana/kibana
    + kibanaHost: https://kibana/kibana
      kibana:
    -   elasticsearchHosts: http://log-storage-master:9200
    +   elasticsearchHosts: https://log-storage-master:9200
        ...
    +   kibanaConfig:
    +     kibana.yml: |
    +       server.ssl:
    +         enabled: true
    +         certificate: /etc/certs/kibana/tls.crt
    +         key: /etc/certs/kibana/tls.key
    +         certificateAuthorities: /etc/certs/ca.crt
    +       elasticsearch:
    +         hosts: ['${ELASTICSEARCH_HOSTS}']
    +         serviceAccountToken: '${ELASTICSEARCH_SERVICEACCOUNTTOKEN}'
    +         ssl.certificateAuthorities: '${ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES:[]}'
    +   extraVolumes:
    +     - name: init-script
    +       configMap:
    +         name: init-kibana-token
    +     - name: root-ca
    +       secret:
    +         secretName: "<release_name>.internal-generated-ca"
    +     - name: kibana-cert
    +       secret:
    +         secretName: kibana-cert
    +   extraVolumeMounts:
    +     - name: root-ca
    +       mountPath: /etc/certs/ca.crt
    +       subPath: tls.crt
    +     - name: kibana-cert
    +       mountPath: /etc/certs/kibana/
    +       readOnly: true
      ...

      filestore:
        ...
    +   tls:
    +     enabled: true
    +     certSecret: filestore-cert
    +     publicCrt: tls.crt
    +     privateKey: tls.key
    ```
