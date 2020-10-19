---
layout: default
title: Assemblyline client
nav_order: 4
parent: User's manual
has_children: false
---

# Assemblyline client
The Assemblyline client library facilitates issuing requests using the REST API to Assemblyline. The client enables you to build scripts to automate various tasks and integrate Assemblyline with other tools or systems.

## Pre-requisites

To install the client you'll need to make sure the you have the following installed:

    # APT/YUM
    libffi-dev
    libssl-dev

    # pypi
    pycryptodome
    requests
    requests[security]
    python-baseconv
    python-socketio[client]
    socketio-client==0.5.7.4

## Using the client

You can instantiate the client using the following snippet of code:

    # The new v4 client will test connection to detect if the server is v3 or v4. You should now use the get_client method.
    from assemblyline_client import get_client
    al_client = get_client("https://localhost:443", auth=('user', 'password'))
    
    # or with an apikey
    
    al_client = get_client("https://localhost:443", apikey=('user', 'key'))
    
    # or with a cert 
    
    al_client = get_client("https://localhost:443", cert='/path/to/cert/file.pem')

    # and if your Assemblyline server is using a self-signed cert

    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify=False)
    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify='/path/to/server.crt')

The Assemblyline client is fully documented in the docstrings so if you use an interactive client like iPython you can use the help feature.

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

### Examples

#### Submit a file

Submitting a file to the system is as simple as passing the file path.

    al_client.submit('/path/to/my/file.txt')

#### Getting a key

To get a key of a given bucket, you simply need to pass it its ID. 

    submission_details = al_client.submission("4nxrpBePQDLH427aA8m3TZ")


#### Using search

You can use the search engine in the client by simply passing a lucene query.

    search_res = al_client.search.submission("submission.submitter:user")

#### Searching different buckets

To search a different bucket simply access a different attribute.

    search_res = al_client.search.file("type:document*")

#### Using search iterator

Instead of using a simple search and getting a page of results, you can use the search iterator to go through all results.

    for submission in al_client.search.stream.submission("submission.submitter:user"):
        # It only return the indexed fields if you want the full thing you need to go get it
        full_submission = al_client.submission(submission['submission.sid'])

        # Then do stuff with full submission (print for example)
        print(full_submission)

#### Using search parameters

##### Version 3
You can pass search parameters for any given query. The following example shows a Lucene facet search to get the top users submitting to a server.

    kwargs = {'facet':'on', 'facet.field':'submission.submitter', 'facet.sort':'count', 'facet.limit':50, 'rows':0}  # rows=0 so that only facet results return
    c.search.submission('times.submitted:[NOW-7DAYS TO NOW]', **kwargs)

##### Version 4
Version 4 server will support facet query out of the box, no need to learn the Lucene faceting syntax.
    
    c.search.facet.submission('submission.submitter', query='times.submitted:[NOW-7DAYS TO NOW]')

#### Listen for message instead of querying for data

You can listen on the different message queues and execute a callback on each message.

    def callback(callback_data):
        print callback_data

    al_client.socketio.listen_on_dashboard_messages(callback)

**NOTE**: Depending on the volume of data, you might process a ton of messages!
