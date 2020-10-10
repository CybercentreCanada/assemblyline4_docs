---
layout: default
title: Deployment types
parent: Installation
has_children: true
has_toc: false
nav_order: 1
---

# Configuration
{: .no_toc }

---

Configuring the Assemblyline 4 system is fairly easy if you know where to look. This section will detail where to look and what to do in order to configure Assemblyline 4 to fit your needs.

Assemblyline support two deployments type; single host which we call "Appliance" and multi-host or "Cluster". They are the same in terms of analysis capabilities however a cluster deployment can be scaled to scan multiple millions of files per day and offer redundancy and fail over. If you are deploying in the cloud a cluster will be easier to deploy (using cloud kubenetes offerings), however in a lab or to support a incident response team any powerful computer will be able to process thoulsand of files a day.

Keep in mind that you will need extra host for running external resources such as anti-virus products, or sandboxes (such as [Cuckoo Sandbox](https://cuckoosandbox.org/)). Theses complementary products are not mandatory but will greatly complement the static analysis and file extraction performed by Assemblyline.

<img src="./images/dep_types.png" width="716">

See [Appliance](./deployment/appliance.html) or [Cluster](./deployment/cluster.html) installation instructions.