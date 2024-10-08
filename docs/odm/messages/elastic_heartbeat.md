[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
# ElasticMessage
Model of Elasticsearch Heartbeat Message

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| msg | [Heartbeat](/assemblyline4_docs/odm/messages/elastic_heartbeat/#heartbeat) | Heartbeat message for elasticsearch | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| msg_loader | Enum | Loader class for message<br>Supported values are:<br>`"assemblyline.odm.messages.elastic_heartbeat.ElasticMessage"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `assemblyline.odm.messages.elastic_heartbeat.ElasticMessage` |
| msg_type | Enum | Type of message<br>Supported values are:<br>`"ElasticHeartbeat"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `ElasticHeartbeat` |
| sender | Keyword | Sender of message | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
## Heartbeat
Heartbeat Model for Elasticsearch

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| instances | Integer | Number of Elasticsearch instances with assigned shards | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| unassigned_shards | Integer | Number of unassigned shards in the cluster | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| request_time | Float | Time to load shard metrics | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| shard_sizes | List [[IndexData](/assemblyline4_docs/odm/messages/elastic_heartbeat/#indexdata)] | Information about each index | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
### IndexData
Information about an elasticsearch shard

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| name | Keyword | None | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| shard_size | Integer | None | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


