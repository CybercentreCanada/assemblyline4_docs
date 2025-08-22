# File storage section

The file storage configuration section (`filestore:`) of the configuration file contains URLs to the different filestores and cachestore used by Assemblyline.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Filestore section configuration example"
    ```yaml
    ...
    # Assemblyline uses a multistage file storing system. When multiple filestores are defined for
    # a single type, Assemblyline will save to all levels at once when adding files but when
    # retrieving file will try one level at the time in order until it finds the file.
    #
    # This allows you to have different retention schedule on the different levels and have faster
    # filestore store only files that are currently scanning in the system but slower ones to keep
    # more files but to look them up less often.

    filestore:
      # List of filestores used for malware archive
      archive:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-archive&use_ssl=False

      # List of URLs to connect to the cache filestore
      cache:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-cache&use_ssl=False

      # List of URLs to connect to the data filestore
      storage:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-storage&use_ssl=False
    ...
    ```

## Supported Transports

All transport connection strings are composed of:

- protocol
- host
- port
- basic authentication (username, password)

!!! info "Example"
    `{protocol}://{username}:{password}@{host}:{port}/`

For every transport protocol, there may be a specific set of parameters for Assemblyline

### Azure

-   allow_directory_access: `boolean`
-   client_id: `string`
-   client_secret: `string`
-   connection_attempts: `integer`
-   tenant_id: `string`
-   use_default_credentials: `boolean`

    !!! info "Example"
        `azure://assemblyline.blob.core.windows.net/?allow_directory_access=false&client_id=&client_secret=&connection_attempts=&tenant_id=&use_default_credentials=false`

### FTP

-   use_tls: `boolean`

    !!! info "Example"
        `ftp://assemblyline:21/?use_tls=`

### HTTP

-   pki: `string`
-   verify: `boolean` | `string`

    !!! info "Example"
        ```plain
        http://assemblyline/?verify=&pki=
        https://assemblyline/?verify=&pki=
        ```

### Local

-   normalize: `boolean`

    !!! info "Example"
        `local://localhost//mnt/al_storage?normalize=`

### S3

-   accesskey: `string`
-   aws_region: `string`
-   boto_defaults: `boolean`
-   connection_attempts: `integer`
-   secretkey: `string`
-   s3_bucket: `string`
-   verify: `string` | `boolean`
-   use_ssl: `boolean`

    !!! info "Example"
        `s3://minio:9000/?s3_bucket=al-storage&aws_region=&accesskey=&secretkey=&boto_defaults=false&connection_attempts=&verify=true&use_ssl`

    !!! note "S3 Certificate Verification"
        For S3-compatible file storage solutions, it is possible to enable verification through the `verify` parameter. The value of the parameter can either be a boolean or a path to the certificate on disk.

        This also assumes that if certificate isn't part of the system certificates, then you'll need to mount it using `coreMounts` & `coreVolumes` and set `verify` to the path of the CA specified in the mount.
        You would also need to modify `configuration.core.scaler.service_defaults.mounts` to ensure privileged services have access to those certificates as well.

### SFTP

-   private_key: `string`
-   private_key_pass: `string`
-   validate_host: `boolean`

    !!! info "Example"
        `sftp://localhost:22/?private_key=&private_key_pass=&validate_host=false`

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
