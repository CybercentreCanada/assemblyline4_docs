# Alert Monitoring And Identifying IoCs For Blocking

## Scenario

> “I want to be able to action on IoCs that Assemblyline has alerted on.”

A common use for Assemblyline is the ability to perform dynamic defence which can involve blocking certain domains/IPs in a firewall

In this exercise, we’re going to look for any alerts containing a network IoC and task that IoC to be blocked (or in this case, just print the IoC that you’re blocking to the console).

## Expected Results

```text
IoCs to be blocked are printed to console
```

## APIs Involved

=== "REST"
    ```text
    GET /api/v4/search/<index>/
    ```
=== "Python"
    ```python
    Client.search.<index>
    ```

## Solution

```python
# Exercise #4: Alert monitoring and identify IOC for blocking

import os
from assemblyline_client import get_client

AL_HOST = os.getenv('AL_HOST', '<AL URL>')
AL_USER = os.getenv('AL_USER', '<AL user>')
AL_APIKEY = os.getenv('AL_APIKEY', '<AL API key>')

# This is the connection to the Assemblyline client that we will use
client = get_client(f"https://{AL_HOST}:443", apikey=(AL_USER, AL_APIKEY), verify=False)


def block_IOC(ioc: str, ioc_type: str, verdict: str):
    # Fake block IOC function, this would obviously be tailored code for your systems
    print(f"Blocking {ioc_type.upper()}: {ioc} ({verdict})")


# Search through the alert index for alerts with IPs and Domains
# client.search.alert --> /api/v4/search/index/
for alert in client.search.alert('al.ip:* OR al.domain:* OR al.uri:*', fl="al.detailed.*")['items']:
    # Iterate over the IOCs in the alert
    for ioc_type in ['ip', 'domain', 'uri']:
        # Iterate through the different items to check if they should be blocked
        for ioc in alert['al']['detailed'][ioc_type]:
            # Make sure those IOCs are not safe or informational​
            if ioc['verdict'] in ['info', 'safe']:
                continue

            # Block suspicious and malicious IOCs (ie. add to FW rules)​
            block_IOC(ioc=ioc['value'], ioc_type=ioc_type, verdict=ioc['verdict'])
```
