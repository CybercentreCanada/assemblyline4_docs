# Collecting Network IoCs

## Scenario

> “I want to collect all the network-related IoCs that Assemblyline was able to extract and store them in a dictionary/mapping. For my use case, I would also want to sort them based on the type of network IoC (ie. Domain, IP, URL)”

## Expected Results

```json
{
    “network.static.ip”: [“172.0.0.1”, ...]
    “network.static.domain”: [“www.google.com”, ...]
    ...
}
```

## APIs Involved

=== "REST"
    ```text
    GET /api/v4/submission/summary/<sid>/
    GET /api/v4/ontology/submission/<sid>/
    ```
=== "Python"
    ```python
    Client.submission.summary(<sid>)
    Client.ontology.submission(<sid>)
    ```

## Solutions

=== "Using Submission API"
    === "Python: Using `assemblyline_client`"
        ```python
        import os
        from pprint import pprint
        from assemblyline_client import get_client

        AL_HOST = os.getenv('AL_HOST', '<AL URL>')
        AL_USER = os.getenv('AL_USER', '<AL user>')
        AL_APIKEY = os.getenv('AL_APIKEY', '<AL API key>')


        # Choose a submission ID that we will use to pull IOCs from
        SID = '<sid>'

        # The result of this exercise will be stored in this variable
        COLLECTED_IOCS = dict()

        # This is the connection to the Assemblyline client that we will use
        client = get_client(f"https://{AL_HOST}:443", apikey=(AL_USER, AL_APIKEY), verify=False)

        # client.submission.summary(<sid>) --> /api/v4/submission/summary/<sid>/
        for tag_name, tag_values in client.submission.summary(SID)['tags']['ioc'].items():
            for tag_value, tag_verdict, is_tag_safelisted, classification in tag_values:
                if tag_name.startswith('network'):
                    # Create the tag category if does not exist
                    COLLECTED_IOCS.setdefault(tag_name, [])

                    # Add the IOC to our list of collected IOCs
                    COLLECTED_IOCS[tag_name].append(tag_value)


        # Now that we have gathered the IOCs, let's print them to the screen
        pprint(COLLECTED_IOCS)
        ```
    === "Python: Using `requests`"
        ```python
        import requests
        import json
        import os
        from pprint import pprint

        headers = {
            "x-user": os.getenv('AL_USER', '<AL user>'),
            "x-apikey": os.getenv('AL_APIKEY', '<AL API key>'),
            "accept": "application/json"
        }

        # Choose a submission ID that we will use to pull IOCs from
        SID = '<sid>'

        # The result of this exercise will be stored in this variable
        COLLECTED_IOCS = dict()

        # This is the connection to the Assemblyline client that we will use
        host = f"https://{os.getenv('AL_HOST', '<AL URL>')}:443"

        # client.submission.summary(<sid>) --> /api/v4/submission/summary/<sid>/
        data = requests.get(f"{host}/api/v4/submission/summary/{SID}/", headers=headers, verify=False).content
        summary = json.loads(data)["api_response"]
        for tag_name, tag_values in summary["tags"]["ioc"].items():
            for tag_value, tag_verdict, is_tag_safelisted, classification in tag_values:
                if tag_name.startswith('network'):
                    # Create the tag category if does not exist
                    COLLECTED_IOCS.setdefault(tag_name, [])

                    # Add the IOC to our list of collected IOCs
                    COLLECTED_IOCS[tag_name].append(tag_value)

        # Now that we have gathered the IOCs, let's print them to the screen
        pprint(COLLECTED_IOCS)
        ```

=== "Using Ontology API"
    === "Python: Using `assemblyline_client`"
        ```python
        import os
        from pprint import pprint
        from assemblyline_client import get_client

        AL_HOST = os.getenv('AL_HOST', '<AL URL>')
        AL_USER = os.getenv('AL_USER', '<AL user>')
        AL_APIKEY = os.getenv('AL_APIKEY', '<AL API key>')


        # Choose a submission ID that we will use to pull IOCs from
        SID = '<sid>'

        # The result of this exercise will be stored in this variable
        COLLECTED_IOCS = dict()

        # This is the connection to the Assemblyline client that we will use
        client = get_client(f"https://{AL_HOST}:443", apikey=(AL_USER, AL_APIKEY), verify=False)

        # client.ontology.submission(<sid>) --> /api/v4/ontology/submission/<sid>/
        for record in client.ontology.submission(SID):
            for tag_name, tag_values in record['results']['tags'].items():
                if tag_name.startswith('network'):
                    # Create the tag category if does not exist
                    COLLECTED_IOCS.setdefault(tag_name, [])

                    # Add the IOC to our list of collected IOCs
                    COLLECTED_IOCS[tag_name].extend(tag_values)

        # Now that we have gathered the IOCs, let's print them to the screen
        pprint(COLLECTED_IOCS)
        ```
    === "Python: Using `requests`"
        ```python
        import requests
        import json
        import os
        from pprint import pprint

        headers = {
            "x-user": os.getenv('AL_USER', '<AL user>'),
            "x-apikey": os.getenv('AL_APIKEY', '<AL API key>'),
            "accept": "application/json"
        }

        # Choose a submission ID that we will use to pull IOCs from
        SID = '<sid>'

        # The result of this exercise will be stored in this variable
        COLLECTED_IOCS = dict()

        # This is the connection to the Assemblyline client that we will use
        host = f"https://{os.getenv('AL_HOST', '<AL URL>')}:443"

        # Option 2: Get IOCs from the ontology API
        # client.ontology.submission(<sid>) --> /api/v4/ontology/submission/<sid>/
        data = requests.get(f"{host}/api/v4/ontology/submission/{SID}/", headers=headers, verify=False).content
        ontology = [json.loads(line) for line in data.splitlines()]
        for record in ontology:
            for tag_name, tag_values in record['results']['tags'].items():
                if tag_name.startswith('network'):
                    # Create the tag category if does not exist
                    COLLECTED_IOCS.setdefault(tag_name, [])

                    # Add the IOC to our list of collected IOCs
                    COLLECTED_IOCS[tag_name].extend(tag_values)

        # Now that we have gathered the IOCs, let's print them to the screen
        pprint(COLLECTED_IOCS)
        ```
