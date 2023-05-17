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
      # List of URLs to connect to the cache filestore
      cache:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-cache&use_ssl=False

      # List of URLs to connect to the data filestore
      storage:
      - s3://al_storage_key:Ch@ngeTh!sPa33w0rd@localhost:9000?s3_bucket=al-storage&use_ssl=False
    ...
    ```

!!! note "S3 Certificate Verification"
    For S3-compatible file storage solutions, it is possible to enable verification through the `verify` parameter. The value of the parameter can either be a boolean or a path to the certificate on disk.

    This also assumes that if certificate isn't part of the system certificates, then you'll need to mount it using `coreMounts` & `coreVolumes` and set `verify` to the path of the CA specified in the mount.
    You would also need to modify `configuration.core.scaler.service_defaults.mounts` to ensure privileged services have access to those certificates as well.

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.
