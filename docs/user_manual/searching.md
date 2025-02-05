# Searching in Assemblyline

Assemblyline provides robust search capabilities within its user interface, allowing users to search for anything stored in its indices. By using the search widget, users can submit queries following the Lucene query syntax, which are then handled by the search engine. The fields available for searching are determined by several Object Data Models (ODMs) captured via Elasticsearch indices.

## Understanding Indices

Elasticsearch indices enable Assemblyline to deduplicate most of the results in the system, which significantly enhances its scalability. Searching through indexed fields is also remarkably fast.

Assemblyline has six primary indices:

- **[Alert](../odm/models/alert.md)**: Allows users to perform detailed searches on alerts to quickly identify and prioritize security incidents, taking into account various attributes such as threat indicators, classification, and timestamps.

- **[File](../odm/models/file.md)**: Allows users to search for specific files within a submission, identify duplicates, and gather context about a file's properties such as its classification, entropy, and observed hash values.

- **[Result](../odm/models/result.md)**: Allows users to search for specific service results, enabling the examination of the analysis performed by various services, including detailed scores, sections, and response data.

- **[Retrohunt](../odm/models/retrohunt.md)**: Allows users to search retrospective threat hunt results from Yara rules applied to previously submitted samples. This facilitates the identification and analysis of new threats based on updated threat intelligence.

- **[Signature](../odm/models/signature.md)**: Allows users to search for service-specific signatures (e.g., YARA rules) and any relevant metadata, including source, statistics, classification, and status.

- **[Submission](../odm/models/submission.md)**: Allows users to manage and track submissions, viewing the files involved, analysis errors, maximum scores, and the lifecycle status of the submission, which provides a holistic view of the analysis process.

You can view all indices and their indexed fields from the `Help > Search Help` menu in your Assemblyline installation.

## Using the Search Interface

### Search Bar

The search bar, located at the top of the user interface, lets you perform searches across all indices.

![A search bar interface, part of the Assemblyline user interface. The search bar is located on a dark background and features a magnifying glass icon on the left side, indicating its function for searching. To the right of the search bar, there are three icons: one for keyboard shortcuts (CTRL+K), another for notifications with a number “12” suggesting there are 12 notifications, and an icon representing the avatar of the logged in user.](./images/search_bar.png){: .center }

### Search Page

Additionally, you can perform searches using the generic Search page.

![A dark-themed search page interface with the word ‘Search’ at the top in white text. Below it, there is a search bar with rounded corners and lighter grey shade. The placeholder text inside the search bar reads ‘Search all indexes…’ in grey. On the right side of the search bar, there is a magnifying glass icon for search and an ‘x’ icon to clear the input field.](./images/search_view.png)

### Search Results

Search results will be displayed across the different indices. The results are categorized by:

- **SUBMISSION**
- **FILE**
- **RESULT**
- **SIGNATURE**
- **ALERT**
- **RETROHUNT**

![A screenshot of a search interface with the word ‘blah’ typed into the search bar. Below the search bar are tabs labeled SUBMISSION, FILE, RESULT, SIGNATURE, ALERT, and RETROHUNT representing various indices, all showing (0) entries. A message box below the tabs states ‘No submissions found!’ and another line reads ‘The query that you ran did not return any results.](./images/searching_across_indices.png)

!!! tip "You must limit your search criteria to a single index. Searching across multiple indices simultaneously (i.e., JOIN queries) is not supported."

This limitation can be mitigated by using the [Assemblyline Client](../../integration/python/) to perform queries on one index and then refine or enrich your search by querying another index.

## Search Examples

### Basic Searches

To familiarize yourself with the indices, use the "Find related results" option from the tags dropdown menu, accessible by right-clicking any tag found throughout Assemblyline.

![Screenshot showing a dropdown menu with options including 'Copy to clipboard', 'Find related results', 'Toggle highlight', and 'Add to safelist'. The 'Find related results' option is highlighted. The dropdown menu can be accessed from right-clicking any tag found throughout Assemblyline.](./images/magnifier.png){: .center }

For example, clicking it on the `av.virus_name` tag (`HEUR/Macro.Downloader.MRAA.Gen`) will generate the following query:
```ruby
result.sections.tags.av.virus_name:"HEUR/Macro.Downloader.MRAA.Gen"
```

### Advanced Searches

Harness the full power of the [Lucene query syntax](https://www.elastic.co/guide/en/kibana/current/lucene-query.html) for more complex searches. Here are a few examples:

```ruby
# Find every result where the ViperMonkey service extracted the IP 10.10.10.10
result.sections.tags.network.static.ip:"10.10.10.10" AND response.service_name:ViperMonkey

# Find all submissions with a score greater than or equal to 2000 in the last two days
max_score:[2000 TO *] AND times.submitted:[now-2d TO now]

# Find all anti-virus results with Emotet in the signature name
result.sections.tags.av.virus_name:*Emotet*
```
Assemblyline supports various search parameters, including wildcards, ranges, and regex. Refer to `Help > Search Help` for comprehensive syntax.

Search queries can also be used with the [Assemblyline Client](../../integration/python) to automate complex tradecraft as new files are processed by the system.

## Autofill Feature

Given the wide range of searchable fields per index, the "autofill" feature can assist in constructing queries. To use autofill, navigate to an index-specific search page, such as "Result" (`/search/result`), and start typing. Autofill will suggest available fields:

![Screenshot of the autofill feature in the Assemblyline application, displaying a dropdown list of suggested search fields that appear when typing 'a' in the search bar. The suggestions include options like 'archive_ts,' 'classification,' 'created,' 'response.extracted.allow_dynamic_recursion,' 'response.extracted.classification,' 'response.extracted.description,' and 'response.extracted.is_section_image.'](./images/autofill_options.png)

For instance, if you wish to query all submissions marked as `TLP:CLEAR` and containing a service that scored greater than 500, you should search within the `Result` index:

![Screenshot of the search results page in the Assemblyline application showing a query for classification:TLP:CLEAR AND result.score:>500. The results include a list of matching entries with columns for Created Time, Verdict, SHA256, File Type, Service, and Classification. Two results are displayed, both marked as 'Malicious' with TLP:CLEAR classification. One entry is a 'archive/zip' file type processed by the 'Extract' service, created 4 days ago, and the other is a 'text/plain' file type processed by the 'NetRep' service, created 12 days ago.](./images/example_query_result.png)
