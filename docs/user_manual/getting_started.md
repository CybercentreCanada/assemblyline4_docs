# Welcome to the Assemblyline User Manual

This documentation is designed to help you get started with using Assemblyline 4, as well as provide detailed information on its features and capabilities.

## Key Concepts

Here's a few key concepts that will help you navigate through the documentation:

### Classification and Sharing

!!! note "If classification enforcement isn't enabled, you can skip this step as you wouldn't see the picker mentioned below."

If your system is configured with a classification scheme, such as TLP, you can select the appropriate classification level for your analysis. This can help ensure that the results of the analysis are shared with the appropriate audience and that sensitive information is handled correctly.

This is an integral part of the submission process, as it allows you to control the visibility and sharing of the analysis results based on the sensitivity of the data being analyzed.

<video controls src="../assets/params_classification.mp4" title="Submission Profiles" autoplay></video>

### Score & Verdict

Each file receives a numeric score that summarizes the risk determined by the services that analyzed it. The score of a submission is determined by the **highest score of any file** extracted during the analysis process.

For example, consider a `zip` archive that scores 0 by itself. If it contains two children files that score 100 and 500 respectively, the submission's overall score will be **500**. You can drill down into the file tree to understand exactly what contributed to each score.

The score maps to a text verdict as follows:

| Score | Verdict |
|---|---|
| -1000 | Safe |
| 0 - 299 | Informative |
| 300 - 699 | Suspicious |
| 700 - 999 | Highly Suspicious |
| ≥ 1000 | Malicious |

For more details, see [Assemblyline Verdicts](../verdicts.md).

### Services

Services are the core components of Assemblyline that perform the analysis on the submitted files. Each service is designed to analyze specific aspects of the file, such as its metadata, behavior, or content. Services can be categorized into different types based on their functionality, such as static analysis, dynamic analysis, reputation checks, and more.

### Heuristics

Heuristics represent patterns or behaviors that a service will raise to draw attention to specific aspects of the file. For example, a heuristic could be raised if a file is packed or obfuscated, which are common techniques used by malware authors to evade detection.

Heuristics can have a score assigned which represents the severity of the heuristic. The higher the score, the more severe the heuristic is considered to be. This can help analysts prioritize which heuristics to investigate first when reviewing the analysis results.

<video controls src="../assets/submission_heuristics.mp4" title="Heuristics" autoplay></video>

### Tags / Indicators of Compromise (IOCs)

Tags represent pieces of information that are extracted from the file and can be used for searching, filtering, and correlation. For example, a tag could be an IP address that was extracted from the file, which could then be used to search for other files that have the same IP address.

![Tags](../assets/submission_tags.png)
