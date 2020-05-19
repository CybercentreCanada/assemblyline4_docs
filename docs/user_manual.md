---
layout: default
title: User's Manual
nav_order: 3
has_children: false
has_toc: false
---

# User manual
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Assemblyline client
The assemblyline-client library facilitates issuing requests to Assemblyline's APIs.

## Pre-requisites

To install the client you'll need to make sure that you have the following libraries installed:

    # APT/YUM
    libffi-dev
    libssl-dev
    
## Installation

Install the Beta version of the client to get the Assemblyline 4 support. The 
client will be smart enough to switch between Assemblyline 3 and 4 support 
depending on which server you connect to.

    pip install assemblyline-client==4.0.0b3

## Using the client

You can instantiate the client by using the following snippet of code:

    # The new v4 client will detect if the server is v3 or v4. 
    # You should now use the get_client method to get an instance of the client.
    
    from assemblyline_client import get_client
    al_client = get_client("https://localhost:443", auth=('user', 'password'))
    
    # or with an apikey
    
    al_client = get_client("https://localhost:443", apikey=('user', 'key'))
    
    # or with a cert 
    
    al_client = get_client("https://localhost:443", cert='/path/to/cert/file.pem')

    # and if your Assemblyline server is using a self-signed cert

    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify=False)
    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify='/path/to/server.crt')

The Assemblyline client is fully documented in the docstrings so if you use an interactive 
client like iPython you can use the help feature.

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

Submitting a file to the system is just as simple as passing the file path

    al_client.submit('/path/to/my/file.txt')

#### Getting a key

To get a key of a given bucket, you simply need to pass it it's ID

    submission_details = al_client.submission("4nxrpBePQDLH427aA8m3TZ")

#### Using search

You can use the search engine in the client by simply passing a Lucene query

    search_res = al_client.search.submission("submission.submitter:user")

#### Using search iterator

Instead of using a straight search and getting a page of results, you can use the search 
iterator to go through all of the results.

    for submission in al_client.search.stream.submission("submission.submitter:user"):
        # It only returns the indexed fields. If you want the full thing you need to go get it
        full_submission = al_client.submission(submission['submission.sid'])

        # Then do stuff with full submission (print for example)
        print(full_submission)

#### Faceting

Version 4 server now supports facet-querying out of the box, no need to learn the 
Lucene-faceting syntax.
    
    al_client.search.facet.submission('submission.submitter', query='times.submitted:[NOW-7DAYS TO NOW]')

#### Listen for message instead of querying for data

You can listen on the different message queues and execute a callback on each message.

    def callback(callback_data):
        print callback_data

    al_client.socketio.listen_on_dashboard_messages(callback)

**NOTE**: Depending on the volume of data, you might process a ton of messages!

## Structures

Messages 

### 