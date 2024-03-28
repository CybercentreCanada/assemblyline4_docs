# Using Retrohunt

## Overview

The Retrohunt feature allows users to scan the historical collection of files in Assemblyline using YARA rules in order to detect early versions of an attack using newer rule sets to understand how an attack has evolved over time.

To start using Retrohunt, make sure you have the correct [configuration](../installation/configuration/retrohunt.md).

If you want to understand how Retrohunt works, a detailed description can be found in the [System Architecture](../overview/architecture.md#yara-back-in-time-retrohunt)

## Retrohunt Interface

Click the "Retrohunt" left navbar link to access the main Retrohunt interface. This page displays all the Retrohunt searches that users have created while taking into consideration their classification. You can filter these searches using your own search parameters, but it does provide quick search actions (on the right side of the search bar) to display jobs that were completed in the last 24h and your own jobs.

![Retrohunt](./images/retrohunt1.png)

## Creating a new Retrohunt search

To create a new search job, click the green "plus" button on the top right of the main interface to open the Create Retrohunt Search page. Enter the following details:

- **Search information**: Provide a description for the search, select which indices to search on (search space) and enter the expiry date (days to live).
- **Maximum file classification**: This property defines the scope of the included files in the search, but note that even with higher classification levels, users with a lower classification will not be able to see all the resulting hits after the search has completed.
- **YARA rule**: When writing a YARA rule, starting to type `rule` will show a snippet that will create a basic rule template. Write a valid rule otherwise submitting the request to create a search will fail.

When you're done, click the "ADD RETROHUNT JOB" and confirm the job to create it. If no errors have occurred, a green snack bar should appear from the bottom and the interface should redirect to its Retrohunt Detail page. That page will show a progress bar denoting the status of the search. After the job has completed, it will show the resulting hits of the search.

![Retrohunt](./images/retrohunt2.png)

## Viewing the hits of a Retrohunt search

The Retrohunt Detail page displays information in a tabular format.

- **Details**: This tab shows the search information and displays the resulting hits of the search. You may reduce the scope of the hits by entering a search query or by clicking on a column in the distribution graphs to add a filter value. Clicking a on table row will open the detail page of that file.
- **YARA Rule**: The entered YARA rule is in this tab to use the full size of the page making it easier to analyze.
- **Errors**: This tab is only accessible to administrators as it shows the warnings and errors that occurred during the Retrohunt search process, which may be used for debugging purposes.

![Retrohunt](./images/retrohunt3.png)

## Repeating a Retrohunt search

The Retrohunt feature offers the ability to repeat completed jobs using the same YARA rule. To repeat a search, click on the "Repeat this Retrohunt search" button on the top right side of the Retrohunt Detail page to open the "Repeat Retrohunt Search" dialog box. You may change the `maximum file classification` and the `days to live`, but note that selecting a lower value will not update that property even though the interface doesn't prevent you from selecting them.

![Retrohunt](./images/retrohunt4.png)
