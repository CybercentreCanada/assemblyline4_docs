# Submitting to Assemblyline

To get started with using Assemblyline for malware analysis, the first step is to submit a file or URL for analysis.

We'll cover the different methods for submitting files and URLs, as well as how to define your analysis parameters to ensure you get the most relevant results from the system.

## Providing the Input

### Submitting Files

The UI supports multiple methods for submitting files for analysis, giving you the flexibility to choose the method that best suits your needs. The most common methods for submitting files are:

#### File Upload

You can drag-and-drop files directly into the dropzone in the UI or click to select files from your local system to upload for analysis under the "File" tab.

#### Using Hashes

You can also submit files for analysis by providing their hashes (MD5, SHA1, or SHA256) if the file has already been analyzed or is known to the system. This can be done under the "Hash/URL" tab.

If your deployment has "File Sources" configured, then the UI will attempt to auto-detect the type of hash you are submitting and provide you with a list of options that the system can use to retrieve the file for analysis. An example of a file source would be one for VirusTotal or Malaware Bazaar.

#### From Clipboard

The UI also supports submitting files directly from your clipboard. At the moment, the system allows for you to paste the contents of your clipboard into the "Raw" tab and this allows you to modify the content before submitting it for analysis.

More commonly, this feature is used to submit snippets of text or code for analysis. For example, you could paste a snippet of PowerShell code into the "Raw" tab and submit it for analysis. The system will treat the pasted content as a file and analyze it accordingly.

### Submitting URLs

The system also allows you to submit URLs for analysis. This can be done under the "Hash/URL" tab by selecting the "URL" option.

When you submit a URL for analysis, the system will provide you with a list of services that are capable of fetching the URL for analysis. Out of the box, this would be the URLDownloader service that is developed by the Assemblyline team.

## Define Your Analysis

### Submission Profiles

### Submission Parameters

#### System Parameters

#### Service Selection

#### Service Parameters
