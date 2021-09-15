# Infrastructure

This section gives you a complete overview of all the different components of Assemblyline so you have a better idea of what they are used for.

## Dependencies

The components listed here are external dependencies used by Assemblyline:

| Dependency | Description | |
|-|-|-|
| Docker | Docker is now at the heart of Assemblyline because all its components are now running as Docker containers. | [https://www.docker.com/](https://www.docker.com/) |
| Kubernetes | For multi-computer installations, Assemblyline uses kubernetes to deploy the different Docker containers and keep them healthy. | [https://kubernetes.io/](https://kubernetes.io/) |
| Helm | Helm is use to easily deploy and maintain our kubernetes instance. | [https://helm.sh/](https://helm.sh/) |
| Elastic Stack | Assemblyline uses the full elastic stack to store results, logs, metrics and APMs. It consists in the following components: | [https://www.elastic.co/elastic-stack](https://www.elastic.co/elastic-stack) |
| | Elasticseach | Elasticsearch is used for storing results, logs and metrics of the system. It also provides search capability to Assemblyline. |
| | Kibana (optional) | Provides dashboards to monitor your Assemblyline cluster |
| | APM (optional) | Gather Application Performance Metrics so we pinpoint potential performance issues with the system and fix them |
| | Filebeat (optional) | Gather all the logs for the different components into Elasticsearch to be displayed in Kibana |
| | Metricbeat (optional) | Gather metrics for the different hosts where the Docker containers are run |
| Redis | We are using Redis for the queueing system, for messaging between the component and as a remote data structure to keep multiple instances of a given component working in sync. | [https://redis.io/](https://redis.io/) |
| Nginx | Nginx is used by Assemblyline as a proxy to give access to the user to the different user facing component: UI, API, Socket Server, Kibana. | [https://www.nginx.com/](https://www.nginx.com/) |
| Minio | For our default file storage, we use Minio which perfectly replicates the Amazon S3 API and is built to work with Kubernetes. | [https://min.io/](https://min.io/) |

## Core Components

The components listed here are Assemblyline made processes that perform various tasks in the system:

| Core components | Description | Link |
|-----------------|-------------|----|
| Alerter | Create alerts for the different submissions in the system. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/alerter) |
| Dispatcher | Route the files in the system while a submission is tacking place. Make sure all files during a submission are completed by all required services. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/dispatching) |
| Expiry | Delete submissions and their results when their TTL expires. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/expiry) |
| Frontend | Provides the user interface to interact with Assemblyline. | [Source code](https://github.com/CybercentreCanada/assemblyline-ui-frontend) |
| Ingester | Move ingested files from the priority queues to the processing queues. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/ingester) |
| Metrics Aggregator | Aggregate metrics of the different components in the system to save them into an ELK stack. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/metrics) |
| Metrics Heartbeat | Provide live metrics in the system for the dashboard. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/metrics) |
| Scaler | Spin up and down services in the system depending on the load. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/scaler) |
| Statistics Aggregator | Generate daily statistics about signatures and heuristics. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/metrics) |
| Updater | Make sure the different services get their latest update files. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/updater) |
| Workflow | Run the different work flows in the system and apply their labels, priority and status. | [Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/workflow) |
| Service Server | Provides an API for services to get tasks and post their results. | [Source code](https://github.com/CybercentreCanada/assemblyline-service-server) |
| UI / Socket Server | Provides the APIs and a socket.io interface to interact with Assemblyline. | [Source code](https://github.com/CybercentreCanada/assemblyline-ui) |

## Service interfaces

The interfaces listed here are used by Assemblyline's services to process files, generate results and communicate back to the Service Server:

| Service Interface | Description | Link |
|-------------------|-------------|------|
| Python 2 Compatibility Library | A library that gives services python 2 compatibility. | [Source code](https://github.com/CybercentreCanada/assemblyline-v4-p2compat) |
| Result | Class used by service to generate results in the system | [Source code](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/result.py) |
| Service Base | Base Assemblyline service class | [Source code](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/base.py) |
| Service Request | Class that definines a request to scan a file for a given service | [Source code](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/request.py) |
| Task Handler | A python wrapper that communicates with the service server to get task, download files and publish results. It communicates with the service via named pipes so the service can execute the received tasks. | [Source code](https://github.com/CybercentreCanada/assemblyline-service-client/tree/master/assemblyline_service_client) |
