# Ingest Files With The Ingest API

## Scenario

> “I want to be able to automate ingestion from a host-based sensor to submit files to Assemblyline and send the parsed results to a database for long-term use.”

Some notes on the Ingest API & submission tracking:

- The Ingest API is an asynchronous API, unlike the Submit API, intended to be a “fire-and-forget” endpoint for submitting files.
- However, at some point, we’d like to be able to get the results from our submissions. We could poll the Search API periodically looking for our submissions, but this puts unnecessary strain on Assemblyline (imagine if 100+ systems were doing this at the same time!).
- Fortunately, the API supports passing in an argument to specify a notification queue name, which generates a new notification queue. The queue can be used to check and retrieve the results from the files submitted under this queue.

## Expected Results

```
A script for ingesting the files into Assemblyline and sets a notification queue

A script for listening on a notification queue for new results to come in
```

## APIs Involved
=== "REST"
    ```
    POST /api/v4/ingest/
    GET /api/v4/ingest/get_message_list/<notification_queue>/
    ```
=== "Python"
    ```python
    Client.ingest()
    Client.ingest.get_message_list(<notification_queue>)
    ```

## Solution

!!! info "The sender made sure to include the `“file_path”` in the metadata so that the receiver knows which files have finished being analyzed."

=== "Sender"
    ```python
    import os
    from assemblyline_client import get_client

    AL_HOST = os.getenv('AL_HOST', '<AL URL>')
    AL_USER = os.getenv('AL_USER', '<AL user>')
    AL_APIKEY = os.getenv('AL_APIKEY', '<AL API key>')

    # Files within this directory will be ingested to Assemblyline
    #  ** Let's just reuse the files from exercise 2
    INGEST_DIR = '/tmp/ex2'
    files_to_scan = list()
    for root, _, files in os.walk(INGEST_DIR):
        for file in files:
            files_to_scan.append(os.path.join(root, file))

    # This is the connection to the Assemblyline client that we will use
    client = get_client(f"https://{AL_HOST}:443", apikey=(AL_USER, AL_APIKEY), verify=False)

    # Submission parameters
    #  ** NOTE: This is only provided as an example you should not use it to get the user's default values.
    #           It's just here to show you that you can alter any submission parameters
    submission_params = {
        'classification': 'TLP:C',                                      # classification
        'description': "I don't wanna use the user's default values!",  # description
        'deep_scan': False,                                             # activate deep scan mode
        'priority': 1000,              # queue priority (the higher the number, the higher the priority)
        'ignore_cache': False,         # ignore system cache
        'services': {
            'selected': [              # selected service list (override user profile)
                'Extract',
                'Safelist'
            ]
        },
        'service_spec': {               # provide a service parameter
            'Extract': {
                'password': 'password'
            }
        }
    }

    # This is the queue that will be use to communicate between your sender and receiver process
    # ** please make it unique for you so you don't receive messages from others!
    NOTIFICATION_QUEUE_NAME = "CHANGE_THIS"

    # Ingest files through Ingest API with a notification queue
    # ** NOTE: you should add the path of the file you just scaned to
    #          the metadata so you can keep track of it
    # client.ingest --> /api/v4/ingest/

    # Ingest all files to scan in Assemblyline
    for file_path in files_to_scan:
        # That's it, just need to send all files in... the receiver will pull the results
        client.ingest(path=file_path, metadata={'file_path': file_path}, nq=NOTIFICATION_QUEUE_NAME)
    ```

=== "Receiver"
    ```python
    import os
    from time import sleep

    from assemblyline_client import get_client

    AL_HOST = os.getenv('AL_HOST', '<AL URL>')
    AL_USER = os.getenv('AL_USER', '<AL user>')
    AL_APIKEY = os.getenv('AL_APIKEY', '<AL API key>')

    # Files within this directory will be ingested to Assemblyline
    #  ** Let's just reuse the files from exercise 2
    INGEST_DIR = '/tmp/ex2'
    files_to_scan = list()
    for root, _, files in os.walk(INGEST_DIR):
        for file in files:
            files_to_scan.append(os.path.join(root, file))

    # This is the connection to the Assemblyline client that we will use
    client = get_client(f"https://{AL_HOST}:443", apikey=(AL_USER, AL_APIKEY), verify=False)

    # This is the queue that will be use to communicate between your sender and receiver process
    # ** You should have changed it in your sender... make it the same here!
    NOTIFICATION_QUEUE_NAME = "CHANGE_THIS"

    # Receive completion messages from the notification queue
    # client.ingest.get_message_list --> /api/v4/ingest/get_message_list/<NOTIFICATION_QUEUE_NAME>/
    while len(files_to_scan) != 0:
        for result in client.ingest.get_message_list(NOTIFICATION_QUEUE_NAME):
            # This is the file we are receiveing result for
            current_file = result['submission']['metadata']['file_path']

            # For each completetion message, pull the result record to get the score
            submission = client.submission(result['submission']['sid'])

            # Print file score to screen
            print(current_file, "=", submission['max_score'])

            # Stop waiting for the file
            files_to_scan.remove(current_file)

        # Otherwise wait for more messages until we're finished
        sleep(1)
    ```
