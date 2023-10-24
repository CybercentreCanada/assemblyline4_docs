# Plugins

The API may be extended via plugins. Plugins may be developed externally to Assemblyline to provide additional
functionality. They must conform to the defined interface for their function.

This section gives you an overview of why plugins exist and what they are used for.

## Why

Plugins are designed to enable interoperability with other systems. This method allows for the de-coupling of system
specific code that is not core to Assemblyline and may not want to be enabled by all users.

These plugins enable a single Assemblyline API to be defined that is able to cleanly query all desired systems, no
matter how obscure.

## Architecture

Plugins are essentially standalone microservices that act as a proxy layer between Assemblyline and the external system.
They exist to translate Assmemblyline specific ontology into queries that the external service will understand. For
example, the builtin external lookup plugin for VirusTotal will take assembyline tags and translate and search for them
in VirusTotal. In this case the Assemblyline tags `network.dynamic.ip` and `network.static.ip` will be mapped to a
VirusTotal search for `ip_address`.

Plugins are run as separate containers and only require incoming ingress connections from the Assemblyline UI API. They
then only require external egress to whatever resources are needed to perform their function.

## Core plugin types

### External Lookups

These plugins enable the querying of Assemblyline data in an external system to then bring back and enrich the
Assembyline Web UI. Use cases of this style of plugin include:

- Looking up IOCs in a private or shared threat intelligence systems (eg. MISP);
- Checking for existance or similar links in public malware analysis systems (eg. VirusTotal and Malware Bazaar);
- Querying internal mirrors and databases

All of this can be achived from within the Assemblyline UI, without having to copy/paste the data and context switch.

The defined interface that these plugins must implement is located in the
[assemblyine-ui plugins/external_lookup/template directory](https://github.com/CybercentreCanada/assemblyline-ui/tree/master/plugins/external_lookup/template).

## Developing a plugin

An example template should be defined for each type of plugin in the
[assemblyine-ui plugins directory](https://github.com/CybercentreCanada/assemblyline-ui/tree/master/plugins/).

The quickest way to develop your own plugin is to simply take a complete copy of this template directory for the type
of plugin you wish to write, and use it as the base of a new repository. In this new repository, modify the `app.py` to
write your functionality. Any Python requirements can be added to the `requirements.txt` file. Tests should then be
written in a `test_app.py` module and any additional test requirements added to a `requirements_tests.txt` file.

The templates also contain a standard Dockerfile for building your container image which will run your app with
`gunicorn`. This Dockerfile can also be modifed as required, as can the basic `gunicorn` config file.

Note: You are not required to use the exact directory structure as the templates provided. These are simply minimal
single module examples. For more complex requirements, you may wish to write a full Python package. The only
requirement is that the query interface is matched.

Once the plugin is written, the Docker image should be built and deployed to your Assemblyline environment. The plugin
can then be added to your Assemblyline configuration. This can be done through Helm by modifying your `values.yaml` or through
Docker by updating your `config.yml`.

## Enabling a plugin

The plugin container should be deployed using either Docker or Kubernetes depending on your operating
environment. Individual plugin configuration can be applied through environment variables, depending on your plugin.
If deployed in Kubernetes with network policies, remember to ensure correct policies are in place.

Once deployed and running, plugins can be enabled by adding their details to the Assemblyline configuration file under
the section: `ui.<plugin_type>`. The required config for each plugin type is defined in
[here](../../../odm/models/config/#externalsource).

Once their configuration has been added, the plugin will be enabled on the next UI reload.

For example, External Lookup plugins can be enabled with the following:

```yaml
ui:
  external_sources:
    - name: <display name for the source>
      url: <full url to the plugin microservice api>
      classification: <optional minimum classification require to access the upstream service>
      max_classification: <optional maximum classification that can be sumbitted to the upstream service>
```

Individual plugin configuration is set in the plugins deployment (eg. through ENV variables in Docker). For example,
the required API key for accessing the VirusTotal API can be set in the plugin setting the `VT_API_KEY` environment
variable in the container.

If deploying to Kubernetes, plugins can be configured and deployed through the Assemblyline Helm chart. All plugin
related fields are stored in the values file under the `uiPlugins` key. Additional custom developed lookup plugins
should be added to the list under: `uiPlugins.lookup.plugins`.

## Included Plugins

### Assemblyline

This plugin can be enabled and configured to query other Assemblyline instances. This may be useful if you
wish to query data from a partner or have multiple instances yourself. All hashes and tags can be queried by default.

#### Container configuration

The following environment variables may be configured:

- API_KEY: Your API key for the service to query with [Default: ""]
- VERIFY: Use secure HTTPS connections [Default: True]
- MAX_LIMIT: Maximum amount of results to return [Default: 100]
- MAX_TIMEOUT: Maximum amount of time to wait for results [Default: 3]
- CLASSIFICATION: The classification of upstream service [Default: TLP:CLEAR]
- URL_BASE: The base URL of the upstream service [Default: https://assemblyline-ui]

### VirusTotal

The following Assembyline data can be queried by default:

- Hashes (MD5, SHA1, SHA256)
- `network.dynamic.domain` and `network.static.domain`
- `network.dynamic.ip` and `network.static.ip`
- `network.dynamic.uri` and `network.static.uri`

#### Container configuration

The following environment variables may be configured:

- VT_API_KEY: Your VirusTotal API key for the service to query with [Default: ""]
- VT_VERIFY: Use secure HTTPS connections [Default: True]
- MAX_TIMEOUT: Maximum amount of time to wait for results [Default: 3]
- CLASSIFICATION: The classification of the data being returned from this service [Default: TLP:CLEAR]
- API_URL: The base URL for the VirusTotal API [Default: https://www.virustotal.com/api/v3]
- FRONTEND_URL: The base URL of the VirusTotal web UI [Default: https://www.virustotal.com/gui/search]

### Malware Bazaar

The following Assembyline data can be queried by default:

- Hashes (MD5, SHA1, SHA256)
- `file.pe.imports.imphash`

#### Container configuration

The following environment variables may be configured:

- MB_VERIFY: Use secure HTTPS connections [Default: True]
- MB_MAX_LIMIT: Maximum amount of results to return [Default: 100]
- MAX_TIMEOUT: Maximum amount of time to wait for results [Default: 3]
- CLASSIFICATION: The classification of the data being returned from this service [Default: TLP:CLEAR]
- API_URL: The base URL for the Malware Bazaar API [Default: https://mb-api.abuse.ch/api/v1]
- FRONTEND_URL: The base URL of the Malware Bazaar web UI [Default: https://bazaar.abuse.ch/browse.php]
