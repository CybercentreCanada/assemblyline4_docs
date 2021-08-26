# Assemblyline Client Library

The assemblyline client library facilitates issuing requests to assemblyline.

## Installing the client

```
install assemblyline_client
```

!!! tip "The client is fully documented in the docstrings so you can use the help feature of ipython or jupyter notebook"

    al_client.search.alert?
    
    Signature: al_client.search.alert(query, *args, **kwargs)
    Docstring:
    Search alerts with a lucene query.

    Required:
    query   : Lucene query. (string)

    Search parameters can be passed as key/value tuples or keyword parameters.

    Returns all results.
    File:      /usr/local/lib/python2.7/dist-packages/assemblyline_client/__init__.py
    Type:      instancemethod

## Connecting to Assemblyline

You can instantiate the client using the following snippet of code:

=== "API Key"

    ``` python
    # Creating an API Key
    # Click on your avatar in the top-right corner and select "Manage account"
    # Scroll down to the bottom to the "Security" section and select "Manage API Keys"
    # Add the API Key name and select access privileges, click Add.
    # Copy it somewhere safe so that you can use it later.
    # Click "Done".

    from assemblyline_client import get_client
    al_client = get_client("https://localhost:443", apikey=('user', 'key'))
    ```
=== "User/Password"

    ``` python
    from assemblyline_client import get_client
    al_client = get_client("https://localhost:443", auth=('user', 'password'))
    ```
=== "Certificate"
 
    ``` python
    from assemblyline_client import get_client
    # and if your assemblyline server is using a self-signed cert
    al_client = get_client("https://localhost:443", cert='/path/to/cert/file.pem')
    ```

??? info "You can provide a certificate or ignore certificate error"
    al_client = get_client(..., verify=False)

    al_client = get_client(..., verify='/path/to/server.crt')

## Examples

### Submit a file or URL

!!! tip "For submitting a URL instead of a file, use the url argument instead of path"

!!! attention "In most cases you want to use the ingest api"
    Ingest: provide a fast, non-blocking method of submitting large amount of files. It also support alert generation.
    Ingest results will typically be analyzed by using a callback (if you need to look at all results) or by monitoring the alerts.

    Submit: high priority, low volume (5 concurent by default, this can be increase slightly in user settings).
    you will need to wait for analysis to complete before submitting more. Useful to support manual analysis.


!!! example "(Optional) Customizing your submission"

    ```python
    # Submission parameters (work for both Ingest and Submit)
    settings = { 
        'classification' : 'TLP:A',     # classification
        'description' : 'Hello world',  # file description
        'name' : 'filename',            # file name
        'deep_scan' : False,            # activate deep scan mode
        'priority' : 1000,              # queue priority (higher nb is higher priority)
        'ignore_cache' : 'false',       # ignore system cache
        'services' : {                  # selected service list (override user profile)
            'selected' : [
                'Cuckoo','Extract'
            ],
            'resubmit' : [],            # resubmit to these services scores > 500
            'excluded': ['']            # exclude some services
        },
        'service_spec': {'Extract': {'password': 'password'}} # provide a service parameter
    }

    # Adding metadata (such as source of the files or anything you want!)
    my_meta = {
        'my_metadata' : 'value',     # any metadata of your liking
        'my_metadata2' : 'value2'     # any metadata of your liking
    }
    ```

    You can find all parameters in the [SubmissionParams class](https://github.com/CybercentreCanada/assemblyline-base/blob/fc5f8216e7fa59d9421ff626927d9602e5a3430c/assemblyline/odm/models/submission.py#L41).

=== "Ingest"
    ```python
    # The ingest api support two extra functionality over the submit api.
    # 1. alert=True: the system will generate an alert if the score is over 500
    # 2. nq='notification_queue_name': callback each time one of your analysis is completed

    al_client.ingest(path='/pathto/file.txt', nq='nq_name', alert=False, params=settings, metadata=my_data)
    # If you use a notificaton queue you can get your results with:
    ingestid=al_client.ingest.get_message("notification_queue_name")
    ```
=== "Submit"
    ```python
    submissionid = al_client.submit(path='/pathto/file.txt', fname='fname', params=settings, metadata=my_meta)
    ```

#### Submission details
You simply need to pass it it's submission ID
``` python
submission_details = al_client.submission("4nxrpBePQDLH427aA8m3TZ")
```

#### Using search
You can use the search engine in the client by simply passing a lucene query
``` python
search_res = al_client.search.submission("submission.submitter:user")
```

#### Using search iterator
Instead of using a strait search and getting a page of result, you can use the search iterator to go through all results.
``` python
for submission in al_client.search.stream.submission("submission.submitter:user"):
    # It only return the indexed fields if you want the full thing you need to go get it
    full_submission = al_client.submission(submission['submission.sid'])

    # Then do stuff with full submission (print for example)
    print(full_submission)
```

#### Using search parameters
``` python
c.search.facet.submission('submission.submitter', query='times.submitted:[NOW-7DAYS TO NOW]')
```
#### Listen for message instead of querying for data

You can listen on the different message queues and execute a callback on each message.
!!! warning "Depending on the volume of data, you might process a ton of messages!"

    def callback(callback_data):
        print callback_data

    al_client.socketio.listen_on_dashboard_messages(callback)

## Using a configuration file
Rather than passing authentication and server details as parameters in a command line, you can use a configuration file.
This configuration file should be placed at `~/.al/submit.cfg`. A template for this configuration 
file can be found below.
NOTE: You can use `=` or `:` as the delimiter between key and value.
``` python
[auth]
#   Username for Assemblyline account.
user = 
#   There are three methods to authenticate a user account. Choose one:
#   - Password Provided via User Prompt
#       Leave the `password' configuration value below empty.
#   - Password Provided in Configuration File
#       Enter the password for Assemblyline account in plaintext.
password = 
#   - API Key in Configuration File
#       Enter the API key to use in plaintext for the user to login.
#       NOTE: The API key must have WRITE access for INGEST and WRITE+READ for SUBMIT.
apikey = 
# Skip server cert validation. 
# Value can be one of: true, false, yes, no
# If not supplied, default value is: false
insecure = 
[server]
# Method of network transport. 
# If not supplied, default value is: https
transport = 
# Domain of Assemblyline instance.
# If not supplied, default value is: localhost
host = 
# Port to which traffic will be sent.
# If not supplied, default value is: 443
port = 
# Server cert used to connect to server.
cert = 
```

## Submission toolkit
[Assemblyline incident manager](https://github.com/CybercentreCanada/assemblyline-incident-manager) that you can assist with this process.

One key consideration for very large volume of files in a burst is [default sampling values](https://github.com/CybercentreCanada/assemblyline-base/blob/9d4ab5586ff34ae20e3a08e9584776379fc981e9/assemblyline/odm/models/config.py#L377).

You have to keep your ingestion flow at a rate such that the size of the ingestion queue remains lower than the corresponding priority values, otherwise Assemblyline will skip files.
