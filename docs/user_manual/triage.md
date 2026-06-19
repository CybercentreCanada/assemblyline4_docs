# Alert Triage

When submissions have `generate_alert` enabled and trigger defined conditions, alerts are raised to draw attention for further examination.

This is a subset of the submission data that is deemed to be of interest, such as a file that has been flagged as malicious or a heuristic that has been raised.

## Alert Management

To triage alerts in Assemblyline, there is a dedicated interface that allows you to view, filter, and manage alerts with ease.

You can access this interface by clicking on the "Alerts" tab (<svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" ><path d="M18 16v-5c0-3.07-1.64-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68C7.63 5.36 6 7.92 6 11v5l-2 2v1h16v-1zm-5 0h-2v-2h2zm0-4h-2V8h2zm-1 10c1.1 0 2-.9 2-2h-4c0 1.1.89 2 2 2"></path></svg>) in the main navigation menu.

### Navigating the Interface

To familiarize yourself with the Alerts interface, here are some key features to note within the main view.

<video controls src="../assets/triage_alerts_view.mp4" title="Alerts Interface" autoplay></video>

#### Filters

Clicking on the "Filters" button (:material-filter-variant:) will open a panel where you can select the desired filters to apply to the list of alerts.

You can also share filters with other users by clicking on the "Share" button (:material-star:). This will open a panel where you can select which filters you want to share and with whom you want to share them. This can be useful for sharing specific views of the alerts with other analysts or for creating shared views for a team.

<video controls src="../assets/triage_filters.mp4" title="Filters" autoplay></video>


#### Using Workflows

Define automated actions for alerts that meet certain conditions through workflows. For instance, mark all alerts labeled as malicious and containing "invoice" as "PHISHING."

There are two approaches on the Alerts page:

- <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" xmlns="http://www.w3.org/2000/svg"><path d="M19 3c-1.654 0-3 1.346-3 3 0 .502.136.968.354 1.385l-1.116 1.302A3.976 3.976 0 0 0 13 8c-.739 0-1.425.216-2.02.566L9.566 7.152A3.449 3.449 0 0 0 10 5.5C10 3.57 8.43 2 6.5 2S3 3.57 3 5.5 4.57 9 6.5 9c.601 0 1.158-.166 1.652-.434L9.566 9.98A3.972 3.972 0 0 0 9 12c0 .997.38 1.899.985 2.601l-1.692 1.692.025.025A2.962 2.962 0 0 0 7 16c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3c0-.476-.121-.919-.318-1.318l.025.025 1.954-1.954c.421.15.867.247 1.339.247 2.206 0 4-1.794 4-4a3.96 3.96 0 0 0-.439-1.785l1.253-1.462c.364.158.764.247 1.186.247 1.654 0 3-1.346 3-3s-1.346-3-3-3zM7 20a1 1 0 1 1 0-2 1 1 0 0 1 0 2zM5 5.5C5 4.673 5.673 4 6.5 4S8 4.673 8 5.5 7.327 7 6.5 7 5 6.327 5 5.5zm8 8.5c-1.103 0-2-.897-2-2s.897-2 2-2 2 .897 2 2-.897 2-2 2zm6-7a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg>+ **Persistent Workflows**:
    Using the current filter, create a persistent workflow that will be applied to all future matching alerts.
    You can manage these persistent workflows from the  "[Manage Workflows](./#workflow-management)" page.

- <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" xmlns="http://www.w3.org/2000/svg"><path d="M19 3c-1.654 0-3 1.346-3 3 0 .502.136.968.354 1.385l-1.116 1.302A3.976 3.976 0 0 0 13 8c-.739 0-1.425.216-2.02.566L9.566 7.152A3.449 3.449 0 0 0 10 5.5C10 3.57 8.43 2 6.5 2S3 3.57 3 5.5 4.57 9 6.5 9c.601 0 1.158-.166 1.652-.434L9.566 9.98A3.972 3.972 0 0 0 9 12c0 .997.38 1.899.985 2.601l-1.692 1.692.025.025A2.962 2.962 0 0 0 7 16c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3c0-.476-.121-.919-.318-1.318l.025.025 1.954-1.954c.421.15.867.247 1.339.247 2.206 0 4-1.794 4-4a3.96 3.96 0 0 0-.439-1.785l1.253-1.462c.364.158.764.247 1.186.247 1.654 0 3-1.346 3-3s-1.346-3-3-3zM7 20a1 1 0 1 1 0-2 1 1 0 0 1 0 2zM5 5.5C5 4.673 5.673 4 6.5 4S8 4.673 8 5.5 7.327 7 6.5 7 5 6.327 5 5.5zm8 8.5c-1.103 0-2-.897-2-2s.897-2 2-2 2 .897 2 2-.897 2-2 2zm6-7a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg>:material-play: **Ephemeral Workflows**:
    Create a one-time action on the current matching alerts, without saving it as a persistent workflow.
    This can be useful for performing quick actions on a specific set of alerts without having to create a new workflow for it.

Workflows empower you to:

- **Assign Status**: Define the alert status (e.g., MALICIOUS, NON-MALICIOUS).
- **Assign Priority**: Specify the alert's urgency (e.g., LOW, MEDIUM, HIGH).
- **Assign Labels**: Categorize the alert for organized management.

<video controls src="../assets/triage_workflow_actions.mp4" title="Workflow Actions" autoplay></video>

### Alert Details

Clicking on an alert card will open the alert details view, where you can see all the information related to that specific alert.

This includes the classification, verdict, labels, file details, metadata, and indicators.

<div class="grid" markdown>
<div class="grid" markdown>
Alert details cover:

- **Classification**: Alert's security level.
- **Basic Information**: Key data about the alert.
- **Verdict and Labels**: The assessed threat level and relevant tags.
- **File Details**: Information such as filename, type, size, and hashes.
- **Metadata and Indicators**: Additional data points and potential security flags.

The actions that can be performed by triage analysts include:

- :material-briefcase-clock-outline: **View History**: Inspect the alert's change log.
- :material-image-filter-center-focus-weak: **Show All Alerts From Group**: Focus on alerts from the same category.
- :material-clipboard-account-outline: **Take Ownership**: Claim the alert for case management.
- :material-view-carousel-outline: **Go to Related Submission**: Transition to connected submission details.
- <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" xmlns="http://www.w3.org/2000/svg"><path d="M19 3c-1.654 0-3 1.346-3 3 0 .502.136.968.354 1.385l-1.116 1.302A3.976 3.976 0 0 0 13 8c-.739 0-1.425.216-2.02.566L9.566 7.152A3.449 3.449 0 0 0 10 5.5C10 3.57 8.43 2 6.5 2S3 3.57 3 5.5 4.57 9 6.5 9c.601 0 1.158-.166 1.652-.434L9.566 9.98A3.972 3.972 0 0 0 9 12c0 .997.38 1.899.985 2.601l-1.692 1.692.025.025A2.962 2.962 0 0 0 7 16c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3c0-.476-.121-.919-.318-1.318l.025.025 1.954-1.954c.421.15.867.247 1.339.247 2.206 0 4-1.794 4-4a3.96 3.96 0 0 0-.439-1.785l1.253-1.462c.364.158.764.247 1.186.247 1.654 0 3-1.346 3-3s-1.346-3-3-3zM7 20a1 1 0 1 1 0-2 1 1 0 0 1 0 2zM5 5.5C5 4.673 5.673 4 6.5 4S8 4.673 8 5.5 7.327 7 6.5 7 5 6.327 5 5.5zm8 8.5c-1.103 0-2-.897-2-2s.897-2 2-2 2 .897 2 2-.897 2-2 2zm6-7a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg> **Perform a Workflow Action**: Execute predefined actions on group alerts.
- :material-shield-check-outline: **Mark as Non-Malicious**: Label the alert as non-threatening.
- :material-bug-outline: **Mark as Malicious**: Label the alert as a confirmed threat.
</div>
<div class="grid" markdown style="align-content: center">
![Alert Detail](../assets/triage_alert_detail.png)
</div>
</div>

## Workflow Management
