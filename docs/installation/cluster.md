# Cluster installation

## Pre-requisites

A kubernetes cluster close to the "vanilla" kubernetes product such as Rancher, AKS (Azure), EKS (Amazon), GKE (Google).

## Cluster Installation

!!! tip "When deployed from this chart an Assemblyline instance must be in its own namespace"

Installation
------------

More detailed installation instructions, and pre-prepared deployment configurations 
for specific kubernetes environments should be available in the future.

1. Make sure you have an ingress controller, set any values for the
   `ingressAnnotations`, `tlsSecretName`, and `configuration.ui.fqdn` parameters.
2. Make sure you have storage classes appropriate for databases and 
   read-write-multiple mounting. Setup the parameters `redisStorageClass`,
   `updateStorageClass`, `log-storage.volumeClaimTemplate`,
   and `datastore.volumeClaimTemplate` to use them.
3. Decide where you want files stored, set the appropriate URI in 
   the `configuration.filestore.*` fields.
4. Enable/disable/configure logging features, (disabled by default).
5. Create a secret based on this template with appropriate passwords:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: assemblyline-system-passwords
type: Opaque
stringData:
  datastore-password:
  logging-password:
    # If this is the password for backends like azure blob storage, the password may need to be url encoded
    # if it includes non alphanum characters
  filestore-password:
  initial-admin-password:
```

Parameters
----------

See [values.yaml](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml)


