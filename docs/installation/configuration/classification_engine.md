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
<td style="background-color:#2869e6"><text style="color:white;">Appliance deployment</text></td>
<td> edit `$HOME/assemblyline4_beta_4/config/config.yml` </td>
</tr>
<tr>
<td style="background-color:#2869e6"><text style="color:white;">Kubernetes deployment</text></td>
<td> see <a href="https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml"> Default helm chart</a> </td>
</tr>
</table>

<hr>