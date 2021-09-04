# Assemblyline results

## Heuristics
A heuristic is a feature that is detected by the service performing the analysis.

- A heuristic has:
    - id
    - name
    - description
    - score (used to label heuristic as MALICIOUS / SUSPICIOUS / INFO)
    - MITRE Att&ck ID
    - Signatures are often raised under a heuristics to provide more context

Heuristics are tracked by the system to provide statistics on the number of hits and scores. Doing so can help with adjusting well-performing and under-performing heuristics.

![Heuristic](./images/heuristic.png)

Heuristics will be shown in the UI and will be colour-coded based on the level of their maliciousness.

![Heuristic](./images/heuristic2.png)

## Tags
Tags are important metadata extracted from a file.

All tag names follow the same naming convention. They are namespaced to improve organization and searching for specific information in the system

Tags are also all indexed, which allow for blazingly fast results.

`network.static.ip     e.g: All IPs extracted statically will be found with this tag regardless of which service extracted it.`

All tags registered with the system will be listed under the Help menu > Current Configuration page > Tags section of your Assemblyline instance.


## Score
The score of a submission (maliciousness level) is determined by the highest score of any file extracted during the analysis process.

For example let's take a `zip` file. The `zip` file itself might score 0 but if it contained two files, one of which 
scored 100 and the other 500, then the max score of the submission will be 500. It is possible to drill down into the file 
structure to understand exactly what contributed to each score.

### Score meaning 
```
< 0: safe
0-299: informational
300 - 699: suspicious
700 - 999: highly suspicious
>= 1000 malicious
```
The source code for these mappings can be found [here](https://github.com/CybercentreCanada/assemblyline-ui-frontend/blob/a030a6400f56b22e11d38132da45467e7651c0fb/src/helpers/utils.ts#L8-L31).

## Submission Report
The first page that will appear when you view a submission is the submission report. This page is a high-level summary to allow an analyst to decide if it is worth digging deeper.

### General information
At the top of the report you will find important information such as timestamps, the file type detected, file size, maximum score and various hashes representing the file contents.

![General information](./images/report_gi.png)

### Heuristics
Under this section you will find all the heuristics, categorized by maliciousness level and every file associated.

![Heuristics](./images/report_heuristics.png)

### Attribution
This section will provide attribution from Yara signatures (if the actor tag is provided in your rule's metadata) and anti-virus virus names.
For best results, follow the [CCCS standard](https://github.com/CybercentreCanada/CCCS-Yara) when writing Yara rules.

![Attribution](./images/report_attribution.png)

## Submission Details
The "Submission Details" button is located at the top of the submission report.

![Details](./images/report_viewdetails.png)

Submission details will display submission parameters such as which services were selected when the file was submitted 
and submission [metadata](../../CLI/client/#submit-a-file-or-url-for-analysis).

### Extracted File Tree
The extracted file tree section will show a view of all the files that were processed and extracted along with their respective score and file type.

Clicking on the files will reveal Assemblyline's most interesting section the File details page.

![Files](./images/report_files.png)

## File Details
Under the "File Details" section you will find everything about a specific file. Regardless of which submission it came from. 

In the top right corner you will find a series of useful functions

* ![Related submission](./images/icon_related_submission.png) Find all related submissions
* ![File download](./images/icon_download.png) Download file (by default the file will be inserted in the [CaRT format](https://pypi.org/project/cart/)) to prevent accidental self-infection
* ![File viewer](./images/icon_fileviewer.png) File viewer (Ascii, Strings, Hex view) ![Hex view](./images/hex.png)
* ![File resubmittion](./images/icon_resubmit.png) Resubmit the file for analysis

### File Frequency
This section will tell you how many times this file has been seen, along with a first and last seen timestamp. These 
values will be affected by the retention period of the file in the system.

![File frequency](./images/file_freq.png)

### File Tags
This section will include all the tags extracted within this file, grouped by type. This is where you will find IPs, 
URLs and many other IoCs (indicators of compromise) which you can harvest to support your investigation or use to start 
a dynamic action (e.g: issue blocks on your firewalls).

If you click on one of the tags it will highlight which service it came from.

![File tags](./images/file_tags.png)

### Service Results
This section lets you visualize the output of each service along with any heuristics and tags raised. You can also see 
which services were the source of "extracted files" at the end of each service result. The cached file results are 
ignored every time a service is updated; if multiple service result versions are available, they will be shown in a 
dropdown which will let you look at older analysis results.

You can expand the details by clicking on a service result section.

![Result section tags](./images/results_section.png)
