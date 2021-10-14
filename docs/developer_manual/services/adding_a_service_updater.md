
# Adding a Service Updater
This documentation builds on [Developing an Assemblyline service](developing_an_assemblyline_service.md) in the event
where you have a service that's dependent on signatures for analysis (ie. YARA or Suricata rules)

## Build your updater
You will need to subclass the `ServiceUpdater` and implement/override functions as deemed necessary.
Refer to [*ServiceUpdater* class](advanced/service_updater_base.md) for more details.


Let's say the following code is written in `update_server.py` in the root of your service directory:

```python
from assemblyline.odm.models.signature import Signature
from assemblyline_v4_service.updater.updater import ServiceUpdater

class SampleUpdateServer(ServiceUpdater):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def import_update(self, files_sha256, client, source, default_classification): -> None
        # Purpose:  Used to import a set of signatures from a source into Assemblyline for signature management
        # Inputs:
        #   files_sha256:           A list of tuples containing file paths and their respective sha256
        #   client:                 An Assemblyline client used to interact with the API on behalf of a service account
        #   source:                 The name of the source
        #   default_classification: The default classification given to a signature if none is provided

        signatures = []
        for file, _ in files_sha256:
            # Iterate over all the files retrieved by source and upload them as signatures in Assemblyline
            with open(file, 'r') as fh:
                signatures.append(Signature(dict(
                    classification = default_classification,
                    data = fh.read(),
                    name = f'sample_signature{len(signatures)}',
                    source = source_name,
                    status = 'DEPLOYED',
                    type = 'sample'
                )))
        client.signatures.add_update_many(signatures)
        self.log.info(f'Successfully imported {len(signatures)} signatures')
        return

    def is_valid(self, file_path) -> bool:
        # Purpose:  Used to determine if the file associated is 'valid' to be processed as a signature
        # Inputs:
        #   file_path:  Path to a signature file from an external source
        return super().is_valid(file_path) #Returns true always

if __name__ == '__main__':
    with SampleUpdateServer(default_pattern="*.json") as server:
        server.serve_forever()
```

## Add it to the manifest!
In addition to your service manifest, you would append the following:

!!! warning
    The updater dependency needs to be named 'updates' for the service to recognize it as an updater rather than a normal dependency container.

!!! critical Updaters **need** to `run_as_core` which allows them to run at the same level as other core containers.

``` yaml
# Adding your dependency called 'updates' and specify the command the container should run
dependencies:
  updates:
    container:
      allow_internet_access: true
      command: ["python", "-m", "update_server"]
      image: ${REGISTRY}testing/assemblyline-service-sample:latest
      ports: ["5003"]
    run_as_core: True

# Update configuration block
update_config:
  # list of source object from where to fetch files for update and what will be the name of those files on disk
  sources:
    - uri: https://file-examples-com.github.io/uploads/2017/02/file_example_JSON_1kb.json
      name: sample_1kb_file
  # interval in seconds at which the updater dependency runs
  update_interval_seconds: 300
  # Should the downloaded files be used to create signatures in the system
  generates_signatures: true
```
