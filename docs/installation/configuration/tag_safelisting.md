---
layout: default
title: Tag safelisting
parent: Configuration
grand_parent: Installation
nav_order: 5
has_toc: false
---

# Tag Safelisting

If you have a certain tag value that you want to safelist for every service, then 
you want to do the following depending on your setup for Assemblyline 4:

## Where to Look
- If you're running a development instance of Assemblyline 4 where the source code from the assemblyline-base repository
is being used directly then do one of the following:
  - Open the following default file: `assemblyline-base/assemblyline/common/tag_whitelist.yml`
  - Override the default file by putting an updated version of `assemblyline-base/assemblyline/common/tag_whitelist.yml`
  in `/etc/assemblyline/tag_whitelist.yml`
  
- If you're running an instance of Assemblyline 4 where you are only using docker-compose, then do the following:
  - Override the default file used by the container by putting an updated version of `assemblyline-base/assemblyline/common/tag_whitelist.yml`
  in `/etc/assemblyline/tag_whitelist.yml`
  
## What to Do
- Once you have the file open and ready to edit, the YML is setup in the following way (and this is detailed in the comments within the file as well):
```
match:
  <tag-type>:
    - <value>

regex:
  <tag-type>:
    - <regular expression>
 ```
- `match` is where you list values for specific tag types that you want to safelist by using a direct string comparison (`==`)
- `regex` is where you list regular expressions for specific tag types that you want to safelist by using regular expression matching (`.match()`)