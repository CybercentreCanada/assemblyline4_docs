
# Adding a Service Updater
This documentation builds on [Developing an Assemblyline service](developing_an_assemblyline_service.md) in the event
where you have a service that's dependent on signatures (ie. YARA/Suricata/Sigma rules) or on a collection of files
(ie. CSV table, custom parsers) for analysis.

## Build your updater
You will need to subclass the `ServiceUpdater` and implement/override functions as deemed necessary.

Refer to [*ServiceUpdater* class](advanced/service_updater_base.md) for more details.

=== "Datastore Persisted"
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

=== "Storage Disk Persisted"
    Let's say the following code is written in `update_server.py` in the root of your service directory:

    ```python
    from assemblyline.odm.models.signature import Signature
    from assemblyline_v4_service.updater.updater import ServiceUpdater

    class SampleUpdateServer(ServiceUpdater):
        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)

        def import_update(self, files_sha256, client, source, default_classification): -> None
            # Purpose:  Used to import a set of signatures from a source into a reserved directory
            # Inputs:
            #   files_sha256:           A list of tuples containing file paths and their respective sha256
            #   client:                 An Assemblyline client used to interact with the API on behalf of a service account
            #   source:                 The name of the source
            #   default_classification: The default classification given to a signature if none is provided

            # You'll want to write your files to self.latest_updates_dir which should hold all your downloaded files.
            # The contents in this directory will then be used by prepare_output_directory().

            # Organize files by source
            dest_dir = os.path.join(self.latest_updates_dir, source)
            os.makedirs(dest_dir, dirs_exist_ok=True)

            # For every file in this source, move to latest updates directory in a subdirectory labelled by source.
            for file, _ in files_sha256:
              dest_file = os.path.join(dest_dir, os.path.basename(file))
              shutil.move(file, dest_file)
            self.log.info(f"Finished moving {len(files_sha256)} files for {source} to {self.latest_updates_dir}")
            return

        def is_valid(self, file_path) -> bool:
            # Purpose:  Used to determine if the file associated is 'valid' to be processed as a signature
            # Inputs:
            #   file_path:  Path to a signature file from an external source
            return super().is_valid(file_path) #Returns true always

        def prepare_output_directory(self) -> str:
            # Purpose: Prepare your downloaded sources before it's made available to your service.
            # This could be a function where if you need to compile a bunch of files together or to re-organize
            # in a certain directory format, you would call this and keep the original dataset intact.
            # Return:
            #    output_directory: the path of the directory to serve
            output_directory = tempfile.mkdtemp()
            shutil.copytree(self.latest_updates_dir, output_directory, dirs_exist_ok=True)
            return output_directory

    if __name__ == '__main__':
        with SampleUpdateServer(default_pattern="*.json") as server:
            server.serve_forever()
    ```

## Add it to the manifest!
In addition to your service manifest, you would append the following:

!!! warning
    The updater dependency needs to be named `updates` for the service to recognize it as an updater rather than a normal dependency container.

!!! danger "Critical"
    Updaters need to `run_as_core` which allows them to run at the same level as other core containers.

=== "Datastore Persisted"
    This configuration is intended if your update files are signatures to be imported into Assemblying using the Signatures API.

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
=== "Storage Disk Persisted"
    You'll have to indicate that this updater doesn't generate signatures (`generate_signatures: false`) for Assemblyline
    and therefore needs to be persisted to disk rather than using a client to interact with the Signatures API
    === "Temporary Disk"
        This configuration sets up downloaded files to only persist on temporary storage for as long as the container runs.

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
          generates_signatures: false
        ```
    === "Persistent Volume"
        This configuration sets up downloaded files to only persist on a persistent volume.

        !!! note
            This will involve using the `UPDATER_DIR` env to tell the updater that your updates will be stored in a
            specific location.

        ``` yaml
        # Adding your dependency called 'updates' and specify the command the container should run
        dependencies:
          updates:
            container:
              allow_internet_access: true
              command: ["python", "-m", "update_server"]
              image: ${REGISTRY}testing/assemblyline-service-sample:latest
              ports: ["5003"]
              # Overwrite the UPDATER_DIR variables to point to your persisted mountpoint
              environment:
                - name: UPDATER_DIR
                  value: /mnt/persistent_updates/
            # Allocate a storage volume and mount it to your updater's filesystem
            volumes:
              updates:
                mount_path: /mnt/persistent_updates/
                capacity: 5242880
                storage_class: mystorageclass
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
          generates_signatures: false
        ```
