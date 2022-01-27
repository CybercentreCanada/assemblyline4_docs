# Python Client

The Assemblyline python client facilitates issuing requests to Assemblyline.

## Installing the client

```
pip install assemblyline_client
```

## Connecting to Assemblyline

You can instantiate the client by using the following snippet of Python code:

??? info "When connecting to Assemblyline, you can also provide a certificate for SSL or ignore the certificate error"
    ``` python
        al_client = get_client(..., verify='/path/to/server.crt')
    
        al_client = get_client(..., verify=False)
    ```

=== "API Key"
    You will need an [API key](../key_generation).
    ``` python
    from assemblyline_client import get_client
    al_client = get_client("https://yourdomain:443", apikey=('user', 'key'))
    ```
=== "User/Password"

    ``` python
    from assemblyline_client import get_client
    al_client = get_client("https://yourdomain:443", auth=('user', 'password'))
    ```
=== "Certificate"
 
    ``` python
    from assemblyline_client import get_client
    # and if your Assemblyline server is using a self-signed certificate
    al_client = get_client("https://yourdomain:443", cert='/path/to/cert/file.pem')
    ```

??? tip "The client is fully documented in the docstrings so that you can use the 'help' feature of IPython or Jupyter Notebook"
    ``` python
        al_client.search.alert?
        
        Signature: al_client.search.alert(
            query,
            filters=None,
            fl=None,
            offset=0,
            rows=25,
            sort=None,
            timeout=None,
        )
        Docstring:
        Search alerts with a Lucene query.

        Required:
        query   : Lucene query. (string)

        Optional:
        filters : Additional Lucene queries used to filter the data (list of strings)
        fl      : List of fields to return (comma separated string of fields)
        offset  : Offset at which the query items should start (integer)
        rows    : Number of records to return (integer)
        sort    : Field used for sorting with direction (string: ex. 'id desc')
        timeout : Max number of milliseconds the query will run (integer)

        Returns all results.
        File:      /usr/local/lib/python3.7/site-packages/assemblyline_client/v4_client/module/search/__init__.py
        Type:      method
    ```

## Examples

### Submit a file or URL for analysis
There are two methods for sending a file/URL to Assemblyline for analysis: **Ingest** and **Submit**.

!!! attention "In most cases, you want to use the Ingest API via the CLI"
    **Ingest**

    * Provides a fast, non-blocking method of submitting many files
    * Ingest results will typically be analyzed by using a callback (if you need to look at all results) or by monitoring the alerts
    * Supports alert generation

    **Submit**

    * High priority, low volume (5 concurrent submissions by default, this can be increased slightly in the user settings)
    * You will need to wait for analysis to complete before submitting more
    * Useful to support manual analysis

!!! example "(Optional) Customizing your submission"
    !!! tip "Note: Service names are case-sensitive"

    ```python
    # Submission parameters (works for both Ingest and Submit)
    settings = { 
        'classification' : 'TLP:A',     # classification
        'description' : 'Hello world',  # file description
        'name' : 'filename',            # file name
        'deep_scan' : False,            # activate deep scan mode
        'priority' : 1000,              # queue priority (the higher the number, the higher the priority)
        'ignore_cache' : False,         # ignore system cache
        'services' : {                  
            'selected' : [              # selected service list (override user profile)
                'Cuckoo','Extract'
            ],
            'resubmit' : [],            # resubmit to these services if file initially scores > 500
            'excluded': [],             # exclude these services
        },
        'service_spec': {               # provide a service parameter
            'Extract': {
                'password': 'password'
            }
        }
    }

    # Adding metadata (such as the source of the files or anything you want!)
    my_meta = {
        'my_metadata' : 'value',     # any metadata of your liking
        'my_metadata2' : 'value2'     # any metadata of your liking
    }
    ```

    You can find all parameters and their default values in the [SubmissionParams class](https://github.com/CybercentreCanada/assemblyline-base/blob/fc5f8216e7fa59d9421ff626927d9602e5a3430c/assemblyline/odm/models/submission.py#L41).

!!! tip "For submitting a URL instead of a file, use the `url` argument instead of `path`"

=== "Ingest"
    The Ingest API supports three additional functionalities over the Submit API:

    * The ingest API is for high throughput submission (feeding the system)
    * By passing the argument `alert=True`, the system will generate an alert if the score is over 500
    * By passing the argument `nq='notification_queue_name'`, you can use the client to poll a notification queue for a message indicating if the analysis has completed
        * If you don't need to know about when the analysis completes, then you can omit the `nq` argument and ignore the subsequent code that interacts with the notification queue
    
    ```python
    ingest_id = al_client.ingest(path='/pathto/file.txt', nq='my_queue_name', params=settings, metadata=my_meta)

    # If you use a notification queue you can get your asynchronous results with:
    from time import sleep
    message = None
    while True:
        message = al_client.ingest.get_message("my_queue_name")
        if message is None:
            sleep(1)    # Poll every second
        else:
            do something ...        
    ```
=== "Submit"
    ```python
    submit_results = al_client.submit(path='/pathto/file.txt', fname='fname', params=settings, metadata=my_meta)
    ```

#### Submission details
To get the details about a submission, you simply need to pass the client a submission ID (sid)
``` python
submission_details = al_client.submission("4nxrpBePQDLH427aA8m3TZ")
```

### Using search
!!! tip "More details about [Search](../user_manual/searching.md)"

You can use the search engine in the client by simply passing a Lucene query.

In the following example, we want to retrieve the first page of submissions made by `user`:
``` python
search_result = al_client.search.submission("params.submitter:user")
```

#### Using search iterator
Instead of using `search` and getting a page of results, you can use the `search` iterator `stream` to go through all the results.
!!! tip "Streamed results only return indexed fields. If you want the full result, you have to go get it via the client"
``` python
for submission in al_client.search.stream.submission("params.submitter:user"):
    submission_id = submission["sid"]
    full_submission = al_client.submission(submission_id)
```

#### Using search parameters
In the following example, we want to retrieve the first page of submissions that were submitted in the last week, and we only want the submission IDs:
``` python
submission_results = al_client.search.submission('times.submitted:[now-7d TO now]', fl='sid')
```

#### Using facet searching
In the following example, we want to retrieve the users who have made submissions in the last week, and the number of submissions that they have made:
``` python
submission_results = al_client.search.facet.submission('params.submitter', query='times.submitted:[now-7d TO now]')
```

## Using the Command-line Tool
By installing the `assemblyline_client` PIP package, a command-line tool `al-submit` is installed.
In case you don't want to use Python code to interface with the Assemblyline client, you can use this tool instead.
You can view the user options via `al-submit --help`.
??? example "(Optional) Configuration file example"
    Rather than passing authentication and server details as parameters in a command-line, you can use a configuration file.
    This configuration file should be placed at `~/.al/submit.cfg`. A template for this configuration 
    file can be found below.
    NOTE: You can use `=` or `:` as the delimiter between key and value.
    ``` python
    [auth]
    #   Username for the Assemblyline account.
    user = 
    #   There are three methods to authenticate a user account. Choose one:
    #   - Password Provided via User Prompt
    #       Leave the `password' configuration value below empty.
    #   - Password Provided in Configuration File
    #       Enter the password for the Assemblyline account in plaintext.
    password = 
    #   - API Key in Configuration File
    #       Enter the API key to use in plaintext for the user to login.
    #       NOTE: The API key must have WRITE access for INGEST and WRITE+READ for SUBMIT.
    apikey = 
    # Skip server cert validation. 
    # Value can be one of: true, false, yes, no
    # If not supplied, the default value is: false
    insecure = 
    [server]
    # Method of network transport. 
    # If not supplied, the default value is: https
    transport = 
    # Domain of Assemblyline instance.
    # If not supplied, the default value is: localhost
    host = 
    # Port to which traffic will be sent.
    # If not supplied, the default value is: 443
    port = 
    # Server cert used to connect to server.
    cert = 
    ```

## Mass Submission Toolkit
The [Assemblyline Incident Manager](https://github.com/CybercentreCanada/assemblyline-incident-manager) can assist you with this process.

One key consideration for a very large volume of files in a burst is the [default sampling values](https://github.com/CybercentreCanada/assemblyline-base/blob/9d4ab5586ff34ae20e3a08e9584776379fc981e9/assemblyline/odm/models/config.py#L392-L396).

You must keep your ingestion flow at a rate such that the size of the priority ingestion queue remains lower than the corresponding priority queue `sampling_at` values, otherwise, Assemblyline will skip files.
