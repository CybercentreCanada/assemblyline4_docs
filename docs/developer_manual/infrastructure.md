---
layout: default
title: Infrastructure
parent: Developer's manual
has_children: true
has_toc: false
nav_order: 1
---

# Infrastructure
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This section of the documentation will explain you how to setup your environement to make changes to Assemblyline's infrastructure and core components. 

First off, this is the list of components that Assemblyline uses.

## Dependancies

Assemblyline uses two following external components to perform its tasks

### Docker

Docker is now at the heart of Assemblyline because all it's components are now running as docker containers. 

[https://www.docker.com/](https://www.docker.com/)

### Kubernetes

For multi-computer installations, Assemblyline uses kubernetes to deploy the different docker containers and keep them healty.

[https://kubernetes.io/](https://kubernetes.io/)

### Helm

Helm is use to easily deploy and maintain our kubernetes instance.

[https://helm.sh/](https://helm.sh/)

### Elastic Stack

Assemblyline uses the full elastic stack to store results, logs, metrics and APMs. It consists in the following components:

[https://www.elastic.co/elastic-stack](https://www.elastic.co/elastic-stack)

#### Elasticseach 

Elasticsearch is used as for storing results, logs and metrics of the system. It also provides search capability to Assemblyline.

#### Kibana (Optional)

Provides dashboards to monitor your Assemblyline cluster

#### APM (Optional)

Gather Application Performance Metrics so we pinpoint potential performance issues with the system and fix them

#### Filebeat (optional)

Gather all the logs for the different components into Elasticsearch to be displayed in Kibana

#### Metricbeat (optional)

Gather metrics for the different hosts where the docker containers are run 

### Redis

We are using Redis for the queing system, for messaging between the component and as a remote datastructure to keep multiple instances of a given component working in sync.

[https://redis.io/](https://redis.io/)

### Nginx 

Nginx is used by Assemblyline as a proxy to give access to the user to the different user facing component: UI, API, Socket Server, Kibana...

[https://www.nginx.com/](https://www.nginx.com/)

### Minio

For our default file storage, we use minio which perfectly replicates the amazon S3 API and is built to work with kubenetes.

[https://min.io/](https://min.io/)

## Core components

Assemblyline has the following core services that makes it all work together.

### Alerter 

Create alerts for the different submissions in the system.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/alerter)

### Dispatcher

Route the files in the system while a submission is tacking place. Make sure all files during a submission are completed by all required services.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/dispatching)

### Expiry

Delete submissions and their results when their TTL expires.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/expiry)

### Ingester

Move ingested files from the priority queues to the processing queues.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/ingester)

### Metrics

Generates metrics of the different components in the system.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/metrics)

### Scaler

Spin up and down services in the system depending on the load.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/scaler)

### Updater

Make sure the different services get their latest update files.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/updater)

### Workflow

Run the different workflows in the system and apply their labels, priority and status.

[Source code](https://github.com/CybercentreCanada/assemblyline-core/tree/master/assemblyline_core/workflow)

### UI / Socket Server

Provides the User Interface, APIs and a socketio interface to interact with Assemblyline

[Source code](https://github.com/CybercentreCanada/assemblyline-ui)

### Service Server 

Provides an API for services to get tasks and post their results.

[Source code](https://github.com/CybercentreCanada/assemblyline-service-server)
