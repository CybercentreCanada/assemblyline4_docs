---
layout: default
title: Tag safelisting
parent: Configuration
grand_parent: Installation
nav_order: 5
has_toc: false
---

# Tag Safelisting

If you have a certain tag value that you want to safelist (ignore certain value so no tag are created) for every service, then 
you want to do the following.

Your configuration file location will depend on your deployment type:

<table>
<tr>
<td style="background-color:#009c7b"><text style="color:white;">Appliance deployment</text></td>
<td> copy `assemblyline-base/assemblyline/common/tag_whitelist.yml` and update into `/etc/assemblyline/tag_whitelist.yml` </td>
</tr>
<tr>
<td style="background-color:#2869e6"><text style="color:white;">Cluster deployment</text></td>
<td> see <a href="https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/object.yaml">Object file</a> </td>
</tr>
</table>

<hr>
  
  
## Editing the safelist
- Once you have the file open and ready to edit, the YML is setup in the following way (and this is detailed in the comments within the file as well):

- `match` is where you list values for specific tag types that you want to safelist by using a direct string comparison (`==`)
- `regex` is where you list regular expressions for specific tag types that you want to safelist by using regular expression matching (`.match()`)
```
  match:
    <tag-type>:
      - <value>

  regex:
    <tag-type>:
      - <regular expression>
```

### Example
```
data:
  tag_whitelist: |
    match:
      network.dynamic.domain:
        - localhost

      network.static.domain:
        - localhost

    regex:
      network.dynamic.ip:
        - (?:127\.|10\.|192\.168|172\.1[6-9]\.|172\.2[0-9]\.|172\.3[01]\.).*
```