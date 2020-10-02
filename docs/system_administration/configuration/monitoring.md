---
layout: default
title: Monitoring
parent: Configuration
grand_parent: System Administration
nav_order: 3
has_toc: false
---
# ELK Deployment
When you deploy Assemblyline using Helm, you have the option of pointing logs to an existing ELK stack you're running or having Assemblyline create it's own internal ELK for logging.

# Configuration
## Elasticsearch & Kibana
If you already have an ELK you would like to use for logging, set and provide the necessary details below in your values.yaml:
```
internalLogging: false
...
loggingHost: <external_logging_host>
kibanaHost: <external_logging_host/kibana>
loggingUsername: "<external_logging_user>>
loggingTLSVerify: "full"
```

However, if you don't have an existing ELK or would prefer Assemblyline use it's own, then set `internalLogging: true` in your values.yaml.

## Logstash Pipelines
You can write [custom pipelines](https://www.elastic.co/guide/en/logstash/current/pipeline.html) to help enrich your data when passed through Logstash. 

You can set your custom logstash pipeline under `customLogstashPipeline` in your values.yaml.

A simple example using Filebeat, by default there is no custom pipeline:
```
customLogstashPipeline: |
input {
  beats {
    port => 5044
    codec => "json"
  }
}

filter{
  mutate { add_field => {"sent_to_logstash" => "True"} }
}

output {
    elasticsearch{
      hosts => "http://elasticsearch:9200"
      index => "assemblyline-logs"
      codec => "json" 
  }
}
```

# Kibana Dashboards
Within Kibana, there is the ability to use dashboards to visualize your data into one consolidated view to make it easier for monitoring, like a hub.

## Creation
Dashboards are made up of [Visualizations](https://www.elastic.co/guide/en/kibana/current/visualize.html), and these can come in different forms: graphs, metrics, gauges, tables, maps, etc.

Each visualization requires an index pattern to get the data from and setting a date range, this throws all relevant data within the specified timeframe into a bucket to be used by the visualization.

Dashboards can also be imported/exported for use across different ELKs **but** requires dependencies like index patterns for them to function out of the box, otherwise requires editing the dashboard file.

## Navigation
All dashboards give you the ability of filtering your data, similar to what you will find under the Discover tab of Kibana.
This will allow you to filter a certain dashboard based on a query you give.

Example: You have a dashboard that contains a metric that counts all the logs on record. You can filter this dashboard to only display logs where the status is "ERROR" or "WARNING" with a query like:

    log.status: "ERROR" or log.status: "WARNING"
    
<img src="./images/dashboard-example.png" width="725">