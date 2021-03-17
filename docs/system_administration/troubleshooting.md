---
layout: default
title: Troubleshooting
parent: System Administration
nav_order: 3
has_toc: false
---

# Troubleshooting

## Signatures
### Q: I've added my signature sources but nothing shows up when searching the Signature Index, what's going on?!
1. Signature Management
    1. Is the URI for the signature source accessible from the host?
    2. Does the signature source require authentication via credentials, CA, headers, etc.?
        1. Provide the appropriate details and monitor the container's logs for errors
    3. Is the signature source a private Git repo?
        1. Be sure to copy the SSH clone information into the URI field and provide the authentication for the repo. 

2. Service Management
    1. What is the update interval set under the service's update config? (the minimum value is 60)
        1. Set the minimum interval to 60s or forcibly restart the updater container if time is of the essence
    2. Is the host able to reach the docker registry to pull in the image to perform the update?
        1. ```curl https://<docker_registry>/v2/_catalog``` should return a list of repositories.
        2. ```curl https://<docker_registry>/v2/repositories/<image_name>/tags``` should return all tags associated to the image (if found)

3. Updater/Scaler
    1. Is the updater and/or scaler container even running or in an error state?
        1. Tail the containers' logs or view them in Kibana if running the full_appliance or equivalent
        
We will update this page with typical issue and solutions.
Until then, please ask us your questions here: [https://groups.google.com/g/cse-cst-assemblyline](https://groups.google.com/g/cse-cst-assemblyline)