# How-to: Upgrade Service Updater for Assemblyline 4.1+

For this tutorial, we'll take an existing official service that was ported to 4.1 as an example [Sigma 4.0.X.Y](https://github.com/CybercentreCanada/assemblyline-service-sigma/blob/a67ebcf1f3b07da689e63e61e6ee90e18d86499c/sigma_updater.py).

## Where do I start?
As a starting point, you would create an instance of the `ServiceUpdater` class.

We recommend setting a `default_pattern (default: *)` as this is a fallback if a source didn't provide a pattern when searching for acceptable signature files. In Sigma's case, its signatures are typically in .yml format.

```python
from assemblyline_v4_service.updater.updater import ServiceUpdater

class SigmaUpdateServer(ServiceUpdater):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def import_update(self, files_sha256, client, source, default_classification): -> None
        pass

    def is_valid(self, file_path) -> bool:
        return super().is_valid(file_path) #Returns true always

if __name__ == '__main__':
    with SigmaUpdateServer(default_pattern="*.yml") as server:
        server.serve_forever()
```

## What do I need to keep from my old updater?

### *Importing into Assemblyine*

In most cases, the means of importing your signatures into Assemblyline's Signature API will still be needed.
As with the following within the if block, you'll be able to copy-paste that section of code into a `import_update()` (with obvious changes to variables names to match the definition). Failure to implement a `import_update()` will raise a NotImplementedError indicating this is a must for an updater.


Starting at line 259 of sigma_updater.py:
```python

    if files_sha256:
        LOGGER.info("Found new Sigma rule files to process!")
        sigma_importer = SigmaImporter(al_client, logger=LOGGER)
        for source, source_val in files_sha256.items():
            total_imported = 0
            default_classification = source_default_classification[source]
            for file in source_val.keys():
                try:
                    total_imported += sigma_importer.import_file(file, source, default_classification=default_classification)
                except ValueError:
                    LOGGER.warning(f"{file} failed to import due to a Sigma error")
                except ComposerError:
                    LOGGER.warning(f"{file} failed to import due to a YAML-parsing error")
            LOGGER.info(f"{total_imported} signatures were imported for source {source}")

    else:
        LOGGER.info('No new Sigma rule files to process')
```
In the new updater:
```python
from assemblyline.common import forge
from assemblyline_v4_service.updater.updater import ServiceUpdater

from sigma_.sigma_importer import SigmaImporter
from pysigma.exceptions import UnsupportedFeature
from yaml.composer import ComposerError

classification = forge.get_classification()

class SigmaUpdateServer(ServiceUpdater):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def import_update(self, files_sha256, client, source, default_classification=classification.UNRESTRICTED): -> None
        sigma_importer = SigmaImporter(client, logger=self.log)
        total_imported = 0
        for file, _ in files_sha256:
            try:
                total_imported += sigma_importer.import_file(file, source, default_classification)
            except ValueError:
                self.log.warning(f"{file} failed to import due to a Sigma error")
            except ComposerError:
                self.log.warning(f"{file} failed to import due to a YAML-parsing error")
            except UnsupportedFeature as e:
                self.log.warning(f'{file} | {e}')

        self.log.info(f"{total_imported} signatures were imported for source {source}")

    def is_valid(self, file_path) -> bool:
        return super().is_valid(file_path) #Returns true always

if __name__ == '__main__':
    with SigmaUpdateServer(default_pattern="*.yml") as server:
        server.serve_forever()
```

### *Validating signature file (Optional)*
Some updaters might've had a means of validating signature files before adding it as part of the signature collection before import.

In Sigma's case, it called `val_file`, after downloading its signature files by either a URL download or a Git clone. If deemed a valid file for the service, then it would add it as part of the signature collection, otherwise it was ignored.

Starting at line 236 of sigma_updater.py:
```python
            if uri.endswith('.git'):
                files = git_clone_repo(source, previous_update=previous_update)
                for file, sha256 in files:
                    files_sha256.setdefault(source_name, {})
                    if previous_hash.get(source_name, {}).get(file, None) != sha256:
                        try:
                            if val_file(file):
                                files_sha256[source_name][file] = sha256
                            else:
                                LOGGER.warning(f"{file} was not imported due to failed validation")
                        except UnsupportedFeature as e:
                            LOGGER.warning(f"{file} | {e}")

            else:
                files = url_download(source, previous_update=previous_update)
                for file, sha256 in files:
                    files_sha256.setdefault(source_name, {})
                    if previous_hash.get(source_name, {}).get(file, None) != sha256:
                        if val_file(file):
                            files_sha256[source_name][file] = sha256
                        else:
                            LOGGER.warning(f"{file} was not imported due to failed validation")
```

In the new updater, you can override the class' `is_valid()` call to suit your validation needs:
```python
from yaml.composer import ComposerError

from assemblyline.common import forge
from assemblyline_v4_service.updater.updater import ServiceUpdater

from sigma_.sigma_importer import SigmaImporter
from pysigma.pysigma import val_file
from pysigma.exceptions import UnsupportedFeature

classification = forge.get_classification()


class SigmaUpdateServer(ServiceUpdater):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def import_update(self, files_sha256, client, source, default_classification=classification.UNRESTRICTED):
        sigma_importer = SigmaImporter(client, logger=self.log)
        total_imported = 0
        for file, _ in files_sha256:
            try:
                total_imported += sigma_importer.import_file(file, source, default_classification)
            except ValueError:
                self.log.warning(f"{file} failed to import due to a Sigma error")
            except ComposerError:
                self.log.warning(f"{file} failed to import due to a YAML-parsing error")
            except UnsupportedFeature as e:
                self.log.warning(f'{file} | {e}')

        self.log.info(f"{total_imported} signatures were imported for source {source}")

    def is_valid(self, file_path) -> bool:
        try:
            return val_file(file_path)
        except UnsupportedFeature:
            return False


if __name__ == '__main__':
    with SigmaUpdateServer(default_pattern="*.yml") as server:
        server.serve_forever()
```

## Is there anything I need to change for the actual service code?
There would be a need to implement a `load_rules()` since services can use their rules in different ways so standardizing on a method isn't feasible at the moment.

In the case of Sigma:

```python
    def _load_rules(self) -> None:
        temp_list = []
        # Patch source_name into signature and import
        for rule in self.rules_list:
            with open(rule, 'r') as yaml_fh:
                file = yaml_fh.read()
                source_name = rule.split('/')[-2]
                patched_rule = f'{file}\nsignature_source: {source_name}'
                temp_list.append(patched_rule)

        self.log.info(f"Number of rules to be loaded: {len(temp_list)}")
        for rule in temp_list:
            try:
                self.sigma_parser.add_signature(rule)
            except Exception as e:
                self.log.warning(f"{e} | {rule}")

        self.log.info(f"Number of rules successfully loaded: {len(self.sigma_parser.rules)}")
```

Note: After each submission, during a `_cleanup()`, there is a call made to `_download_rules()` which will ask your updater if there's new signatures to retrieve. If there's nothing new, then you will proceed to use the same signature set. Otherwise, the service wil retrieve the new set and attempt to load it into the service. For this reason, there can be the need to implement a `_clear_rules()` to ensure service stability if there was a failure while loading the new set that could have a negative impact on analysis.

## What don't I need to implement and why?
### `do_source_update()`
Considering most of our services have the same pattern of:
 1. Establishing a client for Assemblyline using the service_update_account
 2. Iterating over our source list and using either a URL download or Git clone to retrieve signatures
 3. (Optional) If there are rules to be updated, modify the rules before import to add extra metadata or validation
 4. Importing the rules into Assemblyline

 We decided to streamline that process for most services while giving service writers the ability to override certain aspects as needed.

 The service base contains a set of helper functions for retrieving files during the download process. So you'll notice most services don't need to implement their own `do_source_update()`, they just need to implement an `import_update()`. [Safelist](https://github.com/CybercentreCanada/assemblyline-service-safelist) is an example of a service that is an exception to this rule where it does implement it's own `do_source_update()`.

### `do_local_update()`
This particular function doesn't need to be overridden (but you're free to do so) because it's primary function is retrieve the signature set from Assemblyline and make it accessible to service instances when they ask for signatures.

This is separate from `do_source_update()` as that function is meant to retrieve / check for changes from external sources outside Assemblyline whereas `do_local_update()` is checking for internal changes to the signature set (ie. a change in status from 'DEPLOYED' to 'DISABLED')

We invite you to look at the `ServiceUpdater` code to have a better understanding of how the new service updaters work and how they use the functions you implement (`is_valid`, `import_update`).

## Do I have to change the service manifest for 4.1?
Yes, very much so. We've gone away from considering the service updater as a special dependency that needs it's own config section.

We now consider it just be part of the service's list of dependencies, although we reserve the container name 'updates' to indicate this dependency is an updater to the service.


Service Manifest for 4.0:
```yaml

update_config:
  generates_signatures: true
  method: run
  run_options:
    allow_internet_access: true
    command: ["python", "-m", "sigma_updater"]
    image: ${REGISTRY}cccs/assemblyline-service-sigma:$SERVICE_TAG
  sources:
    - name: sigma
      pattern: .*windows\/.*\.yml
      uri: https://github.com/SigmaHQ/sigma.git
  update_interval_seconds: 21600 # Quarter-day (every 6 hours)
```

will now become in 4.1:
```yaml
dependencies:
  updates:
    container:
      allow_internet_access: true
      command: ["python", "-m", "sigma_.update_server"]
      image: ${REGISTRY}cccs/assemblyline-service-sigma:$SERVICE_TAG
      ports: ["5003"]
    run_as_core: True

update_config:
  generates_signatures: true
  sources:
    - name: sigma
      pattern: .*windows\/.*\.yml
      uri: https://github.com/SigmaHQ/sigma.git
  update_interval_seconds: 21600 # Quarter-day (every 6 hours)
  signature_delimiter: "file"
```

Note: Source management will still be part of the update_config, but container configuration will be moved to a list of dependencies.

In order for updaters to work, they need to be able to communicate with other core components. So you'll need to enable the dependency to be able to `run_as_core`, otherwise this could lead to issues where the updater isn't able to resolve to components like redis and/or Elasticsearch.
