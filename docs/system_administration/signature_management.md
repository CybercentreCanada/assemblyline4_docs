---
layout: default
title: Signature management
parent: System Administration
nav_order: 1
has_toc: false
---

# Source Management:
Adding signature sources to support analysis is very simple and can be done directly through the Assemblyline WebUI.

First, navigate to “Source management” through the navbar:

#### <img src="./images/bar.png" width="725">

## **Adding a source:**
Then, to add a signature source, click on the green icon along the right side of the window:
<img src="./images/source_add.png" width="725">


This will bring the user to the window below:

#### <img src="./images/source_config.png" width="725">

### **Required input**

The following sections are <ins>required</ins> in order to add a signature source to Assemlyline:
*	**URI** 
    - This is the path to your sources. In this case, we will use a github repository.
*	**Source Name**
    - This can be labelled at the user’s discretion. For this example, we have used REVERSING LABS.
    
### **Optional input**

*   **Pattern**
    - The user may add a regex pattern to pull certain file types for a particular service. In this example,
    only `.yara` files contained within a specific folder will be added as signatures.
    - CONFIRM THIS WITH GEOFFREY re: how the includes file works

## **Updating signature sources**

The user may set the frequency with which signature sources are updated in two ways:
*    1) Prior to loading the service into Assemblyline; and
*    2) Once the service has been loaded into Assemblyline
    
#### Option 1 - prior to loading the service

*   The updater can be configured through the service_manifest.yml, which is located in the root directory of each service.
#### <img src="./images/root_folder.png" width="500">
*   The following is an example of the update configuration section of the service_manifest.yml file
#### <img src="./images/yml_config.png" width="550">
*   Click [here](https://cybercentrecanada.github.io/assemblyline4_docs/docs/developer_manual/services/service_manifest.html#update-config) to find explanations for each relevant parameter.

#### Option 2 - once the service has been loaded into Assemblyline

*   First navigate to User/Admin/Services through the navbar:
#### <img src="./images/navbar_services.png" width="725">
*   Click on the relevant service you wish to update
*   Navigate to the "updater" tab
*   Input your preferred frequency (in seconds) into the "Update interval" textbox (as seen below)
#### <img src="./images/updater.png" width="725">

## **Signature state**

There are three different signatures states:

*   **Deployed**:
    -   Deployed rules score according to their rule group:
    -   | implant => 1000 | exploit & tool => 500 | technique => 100 | info => 0 |

**Note:** the above rule groups are part of the CCCS YARA specifications for rule metadata. To learn more about this format, please [click here](https://github.com/CybercentreCanada/CCCS-Yara)
*   **Noisy**:
    -   Noisy rules are reported but do not influence the score of the submission
*   **Disabled**:
    -   Are not run against samples that are submitted
    
#### To change a signature's state:

-   Using the navbar, select signature management
#### <img src="./images/signature_management.png" width="725">
-   search for the type of signature you are looking for
#### <img src="./images/search_yara.png" width="725">
**Note**: You may also search for the signature name by adding "name: example_signature_name"
-   Click on the specific rule, select the "Change State" tab, and choose a new signature state from the drop-down menu
#### <img src="./images/change_state.png" width="725">