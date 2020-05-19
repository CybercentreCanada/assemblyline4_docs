---
layout: default
title: Searching
nav_order: 3
parent: User's manual
has_children: false
---

# Searching
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Database
Assemblyline leverages the powerful capabilities of [Elasticsearch](https://www.elastic.co/) making it possible to search for almost anything.

## Document store
One key concept to understand are indexes of information. This allows Assemblyline to deduplicate most of the results in the system which is why it's able to scale so well. Searching indexed fields is also very fast.

- There are 5 primary indexes
    - Submissions
    - Files
    - Results
    - Alerts
    - Signatures

You can view all indexes and their indexed fields once you have a working Assemblyline under `Help > Search help menu`

## Searching behaviors and limitations

When you search in the UI; it will run your query in all the bucket and return any matching results.


Important 
{: .label .label-red }
You must limit your search criteria to a single bucket; in other words you cannot do join query with information present in two or more different buckets. 

This limitation can be worked around using the rest api by performing queries on one bucket and then enriching or narrowing your search by searching elements in other bucket.

# Searching

One quick way to get familiar with search indexes is to use the magnifier icon included in every tag. Clicking the magnifier icon on <img src="./images/magnifier.png" height="50" width="200"/> will build the following query:

```ruby
result.sections.tags.av.virus_name:"Trojan-Downloader.VBA.Agent"
```

You can also build more complex searches by leveraging the full query syntax. Here are some other examples:


```ruby
#Find every results where the ViperMonkey service extracted the IP 10.10.10.10
result.sections.tags.network.static.ip:"10.10.10.10" AND response.service_name:ViperMonkey

#Find all submission with a score of 2000 and above in the last two days
max_score:[2000 TO *] AND times.submitted:[now-2d TO now]

#Find all anti-virus results with Emotet in the signature name
result.sections.tags.av.virus_name:*Emotet*
```
The system support a wide range of search parameter such as wildcards, ranges and regex. The full syntax range can be found under ```Help > Search Help```

Search queries can also be used with the Assemblyline Client to build powerful tradecraft which will run automatically as new files gets scanned by the system.

[So is there restApi?](./assemblyline_client.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }




