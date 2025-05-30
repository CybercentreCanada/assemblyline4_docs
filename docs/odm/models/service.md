[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
# Service
Service Configuration

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| accepts | Keyword | Regex to accept files as identified by Assemblyline | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `.*` |
| auto_update | Boolean | Should the service be auto-updated? | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| rejects | Keyword | Regex to reject files as identified by Assemblyline | <div style="width:100px">:material-minus-box-outline: Optional</div> | `empty|metadata/.*` |
| category | Keyword | Which category does this service belong to? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `Static Analysis` |
| classification | ClassificationString | Classification of the service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `TLP:C` |
| config | Mapping [String, Any] | Service Configuration | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `{}` |
| description | Text | Description of service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `NA` |
| default_result_classification | ClassificationString | Default classification assigned to service results | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `TLP:C` |
| enabled | Boolean | Is the service enabled (by default)? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| is_external | Boolean | Does this service perform analysis outside of Assemblyline? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| licence_count | Integer | How many licences is the service allowed to use? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `0` |
| min_instances | Integer | The minimum number of service instances. Overrides Scaler's min_instances configuration. | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| max_queue_length | Integer | If more than this many jobs are queued for this service drop those over this limit. 0 is unlimited. | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `0` |
| uses_tags | Boolean | Does this service use tags from other services for analysis? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| uses_tag_scores | Boolean | Does this service use scores of tags from other services for analysis? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| uses_temp_submission_data | Boolean | Does this service use temp data from other services for analysis? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| uses_metadata | Boolean | Does this service use submission metadata for analysis? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| monitored_keys | List [Keyword] | This service watches these temporary keys for changes when partial results are produced. | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| name | Keyword | Name of service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| version | Keyword | Version of service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| privileged | Boolean | Should the service be able to talk to core infrastructure or just service-server for tasking? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| disable_cache | Boolean | Should the result cache be disabled for this service? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| stage | Keyword | Which execution stage does this service run in? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `CORE` |
| submission_params | List [[SubmissionParams](/assemblyline4_docs/odm/models/service/#submissionparams)] | Submission parameters of service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| timeout | Integer | Service task timeout, in seconds | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `60` |
| docker_config | [DockerConfig](/assemblyline4_docs/odm/models/service/#dockerconfig) | Docker configuration for service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| dependencies | Mapping [String, [DependencyConfig](/assemblyline4_docs/odm/models/service/#dependencyconfig)] | Dependency configuration for service | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | See [DependencyConfig](/assemblyline4_docs/odm/models/service/#dependencyconfig) for more details. |
| update_channel | Enum | What channel to watch for service updates?<br>Supported values are:<br>`"beta", "dev", "rc", "stable"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `stable` |
| update_config | [UpdateConfig](/assemblyline4_docs/odm/models/service/#updateconfig) | Update configuration for fetching external resources | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| recursion_prevention | List [Keyword] | List of service names/categories where recursion is prevented. | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
## DependencyConfig
Container's Dependency Configuration

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| container | [DockerConfig](/assemblyline4_docs/odm/models/service/#dockerconfig) | Docker container configuration for dependency | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| volumes | Mapping [String, [PersistentVolume](/assemblyline4_docs/odm/models/service/#persistentvolume)] | Volume configuration for dependency | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | See [PersistentVolume](/assemblyline4_docs/odm/models/service/#persistentvolume) for more details. |
| run_as_core | Boolean | Should this dependency run as other core components? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
### DockerConfig
Docker Container Configuration

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| allow_internet_access | Boolean | Does the container have internet-access? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| command | List [Keyword] | Command to run when container starts up. | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| cpu_cores | Float | CPU allocation | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `1.0` |
| environment | List [[EnvironmentVariable](/assemblyline4_docs/odm/models/service/#environmentvariable)] | Additional environemnt variables for the container | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| image | Keyword | Complete name of the Docker image with tag, may include registry | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| registry_username | Keyword | The username to use when pulling the image | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| registry_password | Keyword | The password or token to use when pulling the image | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| registry_type | Enum | The type of container registry<br>Supported values are:<br>`"docker", "harbor"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `docker` |
| ports | List [Keyword] | What ports of container to expose? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| ram_mb | Integer | Container RAM limit | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `512` |
| ram_mb_min | Integer | Container RAM request | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `256` |
| service_account | Keyword | None | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| labels | List [[EnvironmentVariable](/assemblyline4_docs/odm/models/service/#environmentvariable)] | Additional container labels. | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
#### EnvironmentVariable
Environment Variable Model

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| name | Keyword | Name of Environment Variable | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| value | Keyword | Value of Environment Variable | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
### PersistentVolume
Container's Persistent Volume Configuration

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| mount_path | Keyword | Path into the container to mount volume | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| capacity | Keyword | The amount of storage allocated for volume | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| storage_class | Keyword | Storage class used to create volume | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| access_mode | Enum | Access mode for volume<br>Supported values are:<br>`"ReadWriteMany", "ReadWriteOnce"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `ReadWriteOnce` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
## DockerConfig
Docker Container Configuration

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| allow_internet_access | Boolean | Does the container have internet-access? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| command | List [Keyword] | Command to run when container starts up. | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| cpu_cores | Float | CPU allocation | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `1.0` |
| environment | List [[EnvironmentVariable](/assemblyline4_docs/odm/models/service/#environmentvariable)] | Additional environemnt variables for the container | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| image | Keyword | Complete name of the Docker image with tag, may include registry | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| registry_username | Keyword | The username to use when pulling the image | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| registry_password | Keyword | The password or token to use when pulling the image | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| registry_type | Enum | The type of container registry<br>Supported values are:<br>`"docker", "harbor"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `docker` |
| ports | List [Keyword] | What ports of container to expose? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| ram_mb | Integer | Container RAM limit | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `512` |
| ram_mb_min | Integer | Container RAM request | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `256` |
| service_account | Keyword | None | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| labels | List [[EnvironmentVariable](/assemblyline4_docs/odm/models/service/#environmentvariable)] | Additional container labels. | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
### EnvironmentVariable
Environment Variable Model

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| name | Keyword | Name of Environment Variable | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| value | Keyword | Value of Environment Variable | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
## SubmissionParams
Submission Parameters for Service

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| default | Any | Default value (must match value in `value` field) | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| name | Keyword | Name of parameter | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| type | Enum | Type of parameter<br>Supported values are:<br>`"bool", "int", "list", "str"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| value | Any | Default value (must match value in `default` field) | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| list | Any | List of values if `type: list` | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| hide | Boolean | Should this parameter be hidden? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
## UpdateConfig
Update Configuration for Signatures

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| generates_signatures | Boolean | Does the updater produce signatures? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| sources | List [[UpdateSource](/assemblyline4_docs/odm/models/service/#updatesource)] | List of external sources | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| update_interval_seconds | Integer | Update check interval, in seconds | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| wait_for_update | Boolean | Should the service wait for updates first? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| signature_delimiter | Enum | Delimiter used when given a list of signatures<br>Supported values are:<br>`"comma", "custom", "double_new_line", "file", "new_line", "none", "pipe", "space"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `double_new_line` |
| custom_delimiter | Keyword | Custom delimiter definition | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| default_pattern | Text | Default pattern used for matching files | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `.*` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
### UpdateSource
Update Source Configuration

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| name | Keyword | Name of source | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| password | Keyword | Password used to authenticate with source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| pattern | Keyword | Pattern used to find files of interest from source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| private_key | Keyword | Private key used to authenticate with source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| ca_cert | Keyword | CA cert for source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| ssl_ignore_errors | Boolean | Ignore SSL errors when reaching out to source? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| proxy | Keyword | Proxy server for source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| uri | Keyword | URI to source | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| username | Keyword | Username used to authenticate with source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| headers | List [[EnvironmentVariable](/assemblyline4_docs/odm/models/service/#environmentvariable)] | Headers | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `[]` |
| default_classification | Classification | Default classification used in absence of one defined in files from source | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `TLP:C` |
| use_managed_identity | Boolean | Use managed identity for authentication with Azure DevOps | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| git_branch | Keyword | Branch to checkout from Git repository. | <div style="width:100px">:material-minus-box-outline: Optional</div> | `` |
| sync | Boolean | Synchronize signatures with remote source. Allows system to auto-disable signatures no longer found in source. | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| fetch_method | Enum | Fetch method to be used with source<br>Supported values are:<br>`"GET", "GIT", "POST"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `GET` |
| enabled | Boolean | Is this source active for periodic fetching? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `True` |
| override_classification | Boolean | Should the source's classfication override the signature's self-defined classification, if any? | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| configuration | Mapping [String, Any] | Processing configuration for source | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `{}` |
| update_interval | Integer | Update check interval, in seconds, for this source | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |
| ignore_cache | Boolean | Ignore source caching and forcefully fetch from source | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `False` |
| data | Text | Data that's sent in a POST request (`fetch_method="POST"`) | <div style="width:100px">:material-minus-box-outline: Optional</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
#### EnvironmentVariable
Environment Variable Model

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| name | Keyword | Name of Environment Variable | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| value | Keyword | Value of Environment Variable | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


