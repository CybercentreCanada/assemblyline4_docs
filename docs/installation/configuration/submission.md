# Submission section

The submission configuration section (`submission:`) of the configuration file allows you to modify the parameters off how submissions are handled in the system.

Since this section is quite simple, we will list the default configuration at the same time as we describe the different values.

???+ example "Submission section configuration example"
    ```yaml
    ...
    submission:
      # Maximum amount of extracted files for a submission
      default_max_extracted: 500

      # Maximum amount of supplementary files for a submission
      default_max_supplementary: 500

      # Default amount of days submissions live in the system
      dtl: 30

      # Maximum amount of days submissions live in the system
      max_dtl: 0

      # Maximum extraction depth service can go
      max_extraction_depth: 6

      # Maximum file size allowed in the system
      max_file_size: 104857600

      # Maximum size of each metadata entry
      max_metadata_length: 4096

      # Types of tags to be included in the submission summary in
      # the attribution, behaviour and ioc sectiona.
      tag_types:
        attribution:
        - attribution.actor
        - attribution.campaign
        - attribution.exploit
        - attribution.implant
        - attribution.family
        - attribution.network
        - av.virus_name
        - file.config
        - technique.obfuscation
        behavior:
        - file.behavior
        ioc:
        - network.email.address
        - network.static.ip
        - network.static.domain
        - network.static.uri
        - network.dynamic.ip
        - network.dynamic.domain
        - network.dynamic.uri
    ...
    ```

!!! tip
    Refer to the [changing the configuration file](../config_file/#changing-the-configuration-file) documentation for more detail on where and how to change the configuration of the system.

## Metadata Validation

You can configure the system to enforce metadata validation and presence when performing ingestion and archiving. This is a useful feature if you're looking to harmonize the metadata from different sources under a common scheme.

A lot of the configuration is around the parameters of the [ODM fields](https://github.com/CybercentreCanada/assemblyline-base/blob/master/assemblyline/odm/base.py) that Assemblyline uses internally for it's own data validation, so an example of configuring a field using a regex pattern would look like:

```yaml
validation_type: regex
validation_params:
  validation_regex: ^blee
```

So if you wanted to enforce the presence of a metadata field named `bloo` on submission and the value has to match that pattern, the configuration would be:

```yaml
submission:
  metadata:
    submit:
      bloo:  # Field name
        validation_type: regex
        validation_params:
          validation_regex: ^blee
        required: true # Mandatory field to be set on submission
```

The configuration also supports applying strict metadata enforcement, which means a submitter can't add new metadata that the system isn't aware of relative to the scheme:
```yaml
submission:
  metadata:
    ingest:
      INGEST: # "type" parameter when using Ingest API (default: INGEST)
        epoch:
          validation_type: int
        name:
          validation_type: text
    submit:
      name:
        validation_type: text
  strict_schemes: ['INGEST']
```

In the above example, if you're ingesting files under the `INGEST` type, then you can only set the `epoch` or `name` metadata. If there are any additional fields other than those two then the API will return an error. However, you are able to add additional metadata fields when using the Submit API but the `name` field still has to be a string/text type.
