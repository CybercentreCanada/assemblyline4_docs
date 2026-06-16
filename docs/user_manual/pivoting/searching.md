# Searching

Assemblyline indexes the data it generates during analysis, letting you search across submissions, files, results, signatures, alerts, and retrohunts from a single interface using the [Lucene query syntax](https://www.elastic.co/guide/en/kibana/current/lucene-query.html).

## Search Interface

### Simple Search

The search bar at the top of the UI lets you run a quick search across all indices at once. Results are grouped by index, so you can select a tab to see matches for that index.

<video controls src="assets/searching_general.mp4" title="Search"autoplay loop></video>

### Index-Specific Search

For more focused searches, use the dedicated Search page for each index. You can navigate there by pressing the search icon and selecting the desired index.

You can view all available indices and their searchable fields from **Help > Search Help** in your Assemblyline instance.

An index is a collection of related data that can be searched independently. Assemblyline maintains the following indices:

| Index | What you can search for |
|---|---|
| [Alert](../../odm/models/alert.md) | Alerts raised during analysis — useful for triaging and prioritizing security incidents |
| [File](../../odm/models/file.md) | Files seen across all submissions — search by hash, type, entropy, classification, and more |
| [Result](../../odm/models/result.md) | Service results — search scores, extracted sections, and response data from individual services |
| [Signature](../../odm/models/signature.md) | Service signatures such as YARA rules, including their source, status, and statistics |
| [Submission](../../odm/models/submission.md) | Submission records — track files involved, errors, max scores, and lifecycle status |

Searches are scoped to a single index — cross-index (JOIN) queries are not supported in Elasticsearch.

#### Query Building

When looking at a specific index, the search bar will suggest available field names as you type, making it easier to construct valid queries. You can combine fields, values, wildcards, and ranges to build complex queries.

For instance, let's say I want to find all results where any IP was extracted. I can go to the **Result** search page and run a query like:

```
result.sections.tags.network.static.ip:*
```

<video controls src="assets/searching_specific.mp4" title="Searching for IPs"autoplay loop></video>

## Finding Related Results without Searching

When looking at any tag in the UI, you can right-click on it to access a context menu with the option to "Find related results".

This will automatically generate and run a search query for that specific tag value across the system, allowing you to quickly find all related data without needing to manually construct a search query.

<video controls src="assets/searching_related_results.mp4" title="Find related results"autoplay loop></video>

## Automating Searches

Queries can be run programmatically using the [Assemblyline Client](../../integration/python/). This is useful for automating repetitive lookups or enriching results by chaining queries across multiple indices.
