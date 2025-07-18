[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
# ExpiryMessage
Model of Expiry Heartbeat Message

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| msg | [Heartbeat](/assemblyline4_docs/odm/messages/expiry_heartbeat/#heartbeat) | Hearbeat message | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| msg_loader | Enum | Loader class for message<br>Supported values are:<br>`"assemblyline.odm.messages.expiry_heartbeat.ExpiryMessage"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `assemblyline.odm.messages.expiry_heartbeat.ExpiryMessage` |
| msg_type | Enum | Type of message<br>Supported values are:<br>`"ExpiryHeartbeat"` | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `ExpiryHeartbeat` |
| sender | Keyword | Sender of message | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
## Heartbeat
Heartbeat Model

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| instances | Integer | Number of instances | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| metrics | [Metrics](/assemblyline4_docs/odm/messages/expiry_heartbeat/#metrics) | Expiry metrics | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| queues | [Metrics](/assemblyline4_docs/odm/messages/expiry_heartbeat/#metrics) | Expiry queues | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


[comment]: # (AUTOGENERATED MARKDOWN CONTENT. UPDATES TO ODM DOCUMENTATION SHOULD BE DONE THROUGH ASSEMBLYLINE-BASE REPO!)
### Metrics
Expiry Stats

| Field | Type | Description | Required | Default |
| :--- | :--- | :--- | :--- | :--- |
| alert | Integer | Number of alerts | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| badlist | Integer | Number of badlisted items | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| cached_file | Integer | Number of cached files | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| emptyresult | Integer | Number of empty results | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| error | Integer | Number of errors | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| file | Integer | Number of files | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| filescore | Integer | Number of filscores | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| result | Integer | Number of results | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| retrohunt_hit | Integer | Number of retrohunt hits | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| safelist | Integer | Number of safelisted items | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| submission | Integer | Number of submissions | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| submission_tree | Integer | Number of submission trees | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |
| submission_summary | Integer | Number of submission summaries | <div style="width:100px">:material-checkbox-marked-outline: Yes</div> | `None` |


