---
layout: default
title: Classification engine
parent: Configuration
grand_parent: Installation
nav_order: 4
has_toc: false
---

# Classification engine

Assemblyline has a powerful classification engine which can restrict which user can see submission or results.
You can build a configuration which will fit your needs, once a classification engine is enable in the system user will be asked to select a classification when submitting a file.
Services can also choose the classification of certain results section.

Finally you will be able to assign a classification or group to a user and the system will only show submission and results for which the user has the appropriate classification.

If a file is submitted multiple with different classification; the system will automatically downgrade the file classification to the common denominator.

Your configuration file location will depend on your deployment type:

<table>
<tr>
<td style="background-color:#009c7b"><text style="color:white;">Appliance deployment</text></td>
<td> edit `./config/config.yml` </td>
</tr>
<tr>
<td style="background-color:#2869e6"><text style="color:white;">Cluster deployment</text></td>
<td> see <a href="https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/object.yaml">Object file</a> </td>
</tr>
</table>

<hr>

# Configuration example

    classification: |
    enforce: true
    levels:
      - aliases:
        - UNRESTRICTED
        - UNCLASSIFIED
        - U
        css:
          banner: alert-default
          label: label-default
          text: text-muted
        description: Subject to standard copyright rules, TLP:WHITE information may be distributed without restriction.
        lvl: 100
        name: TLP:WHITE
        short_name: TLP:W
      - aliases: []
        css:
          banner: alert-success
          label: label-success
          text: text-success
        description: Recipients may share TLP:GREEN information with peers and partner organizations
          within their sector or community, but not via publicly accessible channels. Information
          in this category can be circulated widely within a particular community. TLP:GREEN
          information may not be released outside of the community.
        lvl: 110
        name: TLP:GREEN
        short_name: TLP:G
      - aliases:
          - RESTRICTED
        css:
          banner: alert-warning2
          label: label-warning
          text: text-warning2
        description: Recipients may only share TLP:AMBER information with members of their
          own organization and with clients or customers who need to know the information
          to protect themselves or prevent further harm.
        lvl: 120
        name: TLP:AMBER
        short_name: TLP:A
    required:
      - aliases: []
        description: Produced using a commercial tool with limited distribution
        name: COMMERCIAL
        short_name: CMR
    subgroups:
      - aliases: []
        description: Incident response
        name: IR
        short_name: IR
      - aliases: []
        description: Member of Group2
        require_group: Group1
        name: Group2
        short_name: Group2

