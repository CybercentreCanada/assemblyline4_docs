# Submitting a SHA256 for analysis

## Submission
Submitting a SHA256 for analysis is very similar to submitting a URL; it can be done directly using the Assemblyline WebUI. For automation and integration you can use the [REST API](../../integration/python/#submit-a-file-url-or-sha256-for-analysis). Just click on the "URL/SHA256" tab.

![URL/SHA256 submission](./images/submit_hash.png)

### Sharing and Classification
Select the desired classification level or sharing restrictions by clicking on the Classification Banner, provided your system configuration includes TLP or another classification scheme.

### Choosing a SHA256 to scan
Rather than dragging and dropping a file or selecting a file from your local drive, you input the SHA256 that you want to scan by typing/pasting it into the "URL/SHA256 To Scan" text box and clicking "SCAN"!

### An important thing to note about SHA256 submissions in Assemblyline
As a default, submitting a SHA256 hash to Assemblyline for analysis will prompt Assemblyline to check its file store to see if the SHA256 exists for any file that Assemblyline has previously seen. If the file exists, then Assemblyline will resubmit that file for analysis. If the file does not exist, then the submission will either fail or Assemblyline will use an external source (if configured) to query the existence of the hash. If the SHA256 is present on an external source like Malware Bazaar, then Assemblyline will download the file from that source and submit that file for analysis. You can set this up in your deployment configuration under the section[`submission.sha256_sources`](../../odm/models/config/#sha256source).

## Options

Access advanced submission options by clicking the "Tune" icon to open the "Adjust" panel. At the top, a banner indicates the level of customization privileges available to you. Users with the `submission_customize` role have the ability to modify all parameters, given that they understand the severe impact some parameters have on the system if miused.

### Submission Parameters:

- **Description**: Optionally provide a description for the analysis, or leave it blank to accept the default value set by the system.
- **Priority**: Designate the submission's processing priority.
- **Days to live**: Specify how long (in days) the file should be retained in the system.
- **Generate alert**: Decide whether the submission should trigger an alert upon analysis completion.
- **Ignore filtering services**: Opt to bypass any safelisting services.
- **Ignore result cache**: Instruct the system to re-analyze the file, disregarding any recent similar analysis.
- **Ignore recursion prevention**: Remove iteration limits for the submission.
- **Perform deep analysis**: Engage thorough deobfuscation, recommended for confirmed malicious or highly suspicious files.

### Submission Data:

- **Decryption Password**: Quickly input a password for encrypted files, bypassing the need to provide it to individual services.

### Service Parameters:

- **Service categories**: Choose a preset group of services.
- **Specific service**: Manually select individual services for the analysis.
- **Service parameters**: Fine-tune service-specific parameters by expanding their individual menus.

### Submission Metadata:

- **System Metadata**: Fill in required system-generated metadata fields.
- **Extra Metadata**: For those with full customization abilities, all additional metadata fields are editable.

![Submit options](./images/hash_submit_options.png)
