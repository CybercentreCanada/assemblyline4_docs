---
layout: default
title: Configuration
parent: Installation
has_children: true
has_toc: false
nav_order: 2
---

# Configuration
{: .no_toc }

---

Configuring the Assemblyline 4 system is fairly easy if you know where to look. 
This section will detail where to look and what to do in order to configure 
Assemblyline 4 to fit your needs.

## Configuration file location

All system level configuration options are set via a yaml configuration file.
The full specification of the file [is defined here](https://github.com/CybercentreCanada/assemblyline-base/blob/master/assemblyline/odm/models/config.py). 
Where the file is located depends on your deployment.

### Docker 

If your system was deployed via one of our docker-compose setups, the configuration
file is located at `config/config.yml`. An example to set the logging output
as json encoded would look like:

```yaml
core:
  logging:
    log_as_json: true
```

### Kubernetes

If you are deploying in kubernetes via our helm chart the configuration file is 
embedded in the values file passed to helm with the `-f` parameter during install 
or upgrade. The system configuration is under the `configuration` key within 
the values file. An example to set the logging output as json encoded would look like:

```yaml
configuration:
  core:
    logging:
      log_as_json: true
```

When configuration options are specified anywhere they will generally not include
the `configuration:` prefix needed in kubernetes, the value set above would be referred to as 
`core.logging.log_as_json`.
