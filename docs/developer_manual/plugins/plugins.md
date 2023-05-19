# Plugins

The API may be extended via plugins. Plugins may be developed externally to Assemblyline to provide additional
functionality. They must conform to the defined interface for their function.

This section gives you an overview of why plugins exist and what they are used for.

## Why

Plugins are designed to enable interoperability with other systems. This method allows for the de-coupling of system
specific code that is not core to Assemblyline and may not want to be enabled by all users.

These plugins enable a single Assemblyline API to be defined cleanly to query all desired systems, no matter how
obscure.

## Architecture

Plugins are essentially standalone microservices that act as a proxy layer between Assemblyline and the external system.
They exist to transalte Assmemblyline specific ontology into queries that the external service will understand. For
example, the builtin external lookup plugin for VirusTotal will take assembyline tags and translate and search for them
in virustotal. In this case the Assemblyline tags `network.dynamic.ip` and `network.static.ip` will be mapped to a
VirusTotal search for `ip_address`.

Plugins are run as separete containers and only require incoming ingress connections from the Assemblyline UI API. They
then only require external egress to whatever resources are needed to perform their function.

## Core plugin types

### External Lookups

These plugins enable the querying of Assmebyline data in an external system and brining back the results.

The defined interface that these plugins must implement is located in the
[assemblyine-ui plugins/external_lookup/template directory](https://github.com/CybercentreCanada/assemblyline-ui/tree/master/plugins/external_lookup/template).

## Developing a plugin

An example template should be defined for each type of plugin in the
[assemblyine-ui plugins directory](https://github.com/CybercentreCanada/assemblyline-ui/tree/master/plugins/).

The quickest way to develop your own plugin is to simply take a complete copy of this template directory for the type
of plugin you wish to write, and use it as the base of a new repository. In this new repository, modify the `app.py` to
write your functionality. Any python requirements can be added to the `requirements.txt` file. Tests should then be
written in a `test_app.py` module and any additional test requirements added to a `requirements_tests.txt` file.

The templates also contain a standard Dockerfile for building your container image which will run your app with
`gunicorn`. This Dockerfile can also be modifed as required, as can the basic `gunicorn` config file.

Once the plugin is written, the docker image should be built and deployed to your Assemblyline environment. The plugin
can then be added to your assemblyline configuration.

## Enabling a plugin

Once deployed, plugins can be enabled by adding their details to the Assemblyline configuration file under the section:
`ui.plugins.<plugin_type>`. The required config for each plugin type is defined in
[Assemblyline Base](https://github.com/CybercentreCanada/assemblyline-base/blob/master/assemblyline/odm/models/config.py).

Once their configuration has been added, the plugin will be enabled on reload.

For example, External Lookup plugins can be enabled with the following:

```yaml
ui:
  plugins:
    external_sources:
      - name: virustotal
        url: <full url to the plugin microservice>
        classification: <optional minimum classification of the upstream service>
        max_classification: <optional maximum classification that can be sumbitted to the upstream service>
```

Individual plugin configuration is set in the plugins deployment (eg through ENV variables in docker).
