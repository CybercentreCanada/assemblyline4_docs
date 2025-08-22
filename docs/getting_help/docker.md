# Docker

This section contains troubleshooting steps for deploying Assemblyline in a Docker environment.

??? question "Redis is logging: 'WARNING Memory overcommit must be enabled!', what should I do?"
    To fix this issue add `vm.overcommit_memory = 1` to /etc/sysctl.conf and then reboot or run the command `sysctl vm.overcommit_memory=1` for this to take effect. This has to be performed on the host that's running the deployment.
