# Searching
Assemblyline leverages the powerful capabilities of [Elasticsearch](https://www.elastic.co/) which make it possible to 
search for almost anything.

## Document store
One key concept to understand are the indices (buckets) of information. These allow Assemblyline to deduplicate most of the results 
in the system which is a major reason why Assemblyline is able to scale so well. Searching indexed fields is also very fast.

- There are 5 primary indices (buckets)
    - Submissions
    - Files
    - Results
    - Alerts
    - Signatures

You can view all indices and their indexed fields once you have a working Assemblyline under `Help > Search Help` menu.

## Searching behaviors and limitations
When you search for something in the Search Bar at the top of the UI, it will run your query in all the buckets and return any matching results.

!!! tip "You must limit your search criteria to a single bucket; in other words you cannot do JOIN queries with information present in two or more buckets." 

This limitation can be worked around by using the [Assemblyline Client](../CLI/client.md) by performing queries on one bucket and then enriching or narrowing your search by searching for elements in another bucket.

# Searching
One quick way to get familiar with search indices is to use the magnifying glass icon included in every tag. Clicking the magnifying glass icon on ![Searching](./images/magnifier.png) will build the following query:
```ruby
result.sections.tags.av.virus_name:"Trojan-Downloader.VBA.Agent"
```

You can also build more complex searches by leveraging the full query syntax. Here are some other examples:
```ruby
# Find every result where the ViperMonkey service extracted the IP 10.10.10.10
result.sections.tags.network.static.ip:"10.10.10.10" AND response.service_name:ViperMonkey

# Find all submissions with a score greater than or equal to 2000 in the last two days
max_score:[2000 TO *] AND times.submitted:[now-2d TO now]

# Find all anti-virus results with Emotet in the signature name
result.sections.tags.av.virus_name:*Emotet*
```
The system supports a wide range of search parameter such as wildcards, ranges and regex. The full syntax range can be found under ```Help > Search Help```

Search queries can also be used with the [Assemblyline Client](../../integration/python) to build powerful tradecraft which can run automatically as new files get scanned by the system.





