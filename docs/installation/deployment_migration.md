# Docker-Compose → Kubernetes System Migration

This guide assumes you're moving from a vanilla Docker-Compose-based deployment to a Kubernetes deployment.

!!! caution "No processing on either system should be occurring during this migration process"

## Docker-Compose System

### Create a Docker service for backups
Create a `backup` container that has access to the datastore's API and the filestore volume.

You can modify the `docker-compose.yaml` of your deployment type like the following:
```yaml
  # Add a new service called 'backup' to facilitate backing up Assemblyline
  backup:
    build:
      context: privilege-core-image
      args:
        version: ${AL_VERSION}
        registry: ${REGISTRY}
      cache_from:
        - cccs/assemblyline-core:${AL_VERSION}
    image: ${COMPOSE_PROJECT_NAME}_scaler:${AL_VERSION}
    privileged: true
    env_file: [".env"]
    volumes:
      - ${COMPOSE_ROOT}/config/config.yml:/etc/assemblyline/config.yml:ro
      - backup:/mount/backup:rw       # Mount backup volume
      - filestore:/data               # Mount filestore volume

    networks: [core]
    command: "sleep 9999999999d"      # Start up and keep running so we can shell into the container later

volumes:
  ...
  backup:                             # Add a backup volume
```

### Backup Datastore indices

Shell into your `backup` container, open the AL CLI tool, and backup the datastore data to the mount.
=== "System-only"
    !!! info
        The following indices will be backed up:

        `heuristic, service, service_delta, signature, user, user_avatar, user_favorites, user_settings, workflow`

    ```bash
    python -m assemblyline.run.cli
    backup /mount/backup/al_system
    ls /mount/backup/
    ```
=== "Everything"
    !!! info "All indices will be backed up"

    ```bash
    python -m assemblyline.run.cli
    backup /mount/backup/al_alert alert force *
    backup /mount/backup/al_cached_file cached_file force *
    backup /mount/backup/al_emptyresult emptyresult force *
    backup /mount/backup/al_error error force *
    backup /mount/backup/al_file file force *
    backup /mount/backup/al_filescore filescore force *
    backup /mount/backup/al_heuristic heuristic force *
    backup /mount/backup/al_result result force *
    backup /mount/backup/al_safelist safelist force *
    backup /mount/backup/al_service_delta service_delta force *
    backup /mount/backup/al_service service force *
    backup /mount/backup/al_signature signature force *
    backup /mount/backup/al_submission submission force *
    backup /mount/backup/al_submission_summary submission_summary force *
    backup /mount/backup/al_submission_tree submission_tree force *
    backup /mount/backup/al_user_avatar user_avatar force *
    backup /mount/backup/al_user user force *
    backup /mount/backup/al_user_settings user_settings force *
    backup /mount/backup/al_workflow workflow force *
    exit
    ls /mount/backup/
    ```
### Backup Filestore directories
Copy `al-cache` and `al-storage` to the backup volume
```bash
cp -r /data/ /mount/backup/filestore/
```

## Kubernetes System

### Restoring the Datastore

#### Mount your backup using coreVolumes and coreMounts
Edit your `values.yaml` as follows and then perform a `helm upgrade`:
```yaml
coreVolumes:
  - name: assemblyline-backup
    hostPath:
      path: /var/lib/docker/volumes/al_backup/_data
      type: Directory
coreMounts:
  - name: assemblyline-backup
    mountPath: /mount/backup
```

#### Restore Datastore by using the Assemblyline CLI
Shell into one of the core containers (ie. scaler) and perform the following:
=== "System-only"
    ```bash
    python -m assemblyline.run.cli
    restore /mount/backup/al_system
    ```
=== "Everything"
    !!! warning "Not all systems are equal"
        You will only need to restore with a subset of the folders listed below as your deployment isn't guaranteed to
        have data in every index (ie. alerts).

        Performing an `ls /mount/backup` will confirm what you have backed up.

    ```bash
    python -m assemblyline.run.cli
    restore /mount/backup/al_alert alert force *
    restore /mount/backup/al_cached_file cached_file force *
    restore /mount/backup/al_emptyresult emptyresult force *
    restore /mount/backup/al_error error force *
    restore /mount/backup/al_file file force *
    restore /mount/backup/al_filescore filescore force *
    restore /mount/backup/al_heuristic heuristic force *
    restore /mount/backup/al_result result force *
    restore /mount/backup/al_safelist safelist force *
    restore /mount/backup/al_service_delta service_delta force *
    restore /mount/backup/al_service service force *
    restore /mount/backup/al_signature signature force *
    restore /mount/backup/al_submission submission force *
    restore /mount/backup/al_submission_summary submission_summary force *
    restore /mount/backup/al_submission_tree submission_tree force *
    restore /mount/backup/al_user_avatar user_avatar force *
    restore /mount/backup/al_user user force *
    restore /mount/backup/al_user_settings user_settings force *
    restore /mount/backup/al_workflow workflow force *
    exit
    ```

### Migrating the Filestore

#### Edit Filestore Deployment in Kubernetes
Edit deployment and add the following hostpath mount:

```yaml
spec:
  ...
  template:
    ...
    spec:
      containers:
        - name: filestore
          ...
          volumeMounts:
            ...
            - mountPath: /mount/backup
              name: backup
      ...
      volumes:
        ...
        - hostPath:
            path: /var/lib/docker/volumes/al_backup/_data
            type: Directory
          name: backup
```

#### Merge Filestore data with existing volume
Shell into the filestore pod and copy the files from backup:

```bash
cp -r /mount/backup/filestore/* /export/
```

Once transfer is complete, you can remove the `volumeMount` and `volume` relating to the backup from the deployment.

### Migrate System Configuration
You can copy the contents of `config.yml` and place them in the `configuration` key of your `values.yaml`

!!! warning "Some configuration values are oriented towards the Docker-Compose appliance, so change values where applicable."

### Extra Configurations!
If you want to use custom configurations to control tag safelisting or classification, you'll need to create a ConfigMap and
modify your helm deployment to make use of them.

Let's say I want to add the following custom configurations to Kubernetes:
=== "classification.yml"
    ```
    enforce: true
    ```
=== "tag_safelist.yml"
    ```
    match:
      network.dynamic.domain:
        - localhost
    ```

You'll need to create a new file, say `objects.yaml`, with the following:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: assemblyline-extra-config
data:
  tag_safelist: |
    match:
      network.dynamic.domain:
        - localhost
  classification: |
    enforce: true
```
Followed by a `kubectl apply -f objects.yaml -n <al_namespace>`

Once the new ConfigMap has been created in your Kubernetes namespace, you can modify your `values.yaml` as follows:
```yaml
coreEnv:
  - name: CLASSIFICATION_CONFIGMAP
    value: assemblyline-extra-config
  - name: CLASSIFICATION_CONFIGMAP_KEY
    value: classification
coreMounts:
  - name: al-extra-config
    mountPath: /etc/assemblyline/tag_whitelist.yml
    subPath: tag_safelist
    readOnly: true
  - name: al-extra-config
    mountPath: /etc/assemblyline/classification.yml
    subPath: classification
    readOnly: true
coreVolumes:
  - name: al-extra-config
    configMap:
      name: assemblyline-extra-config
```
Followed by a `helm upgrade` to apply the configurations.
