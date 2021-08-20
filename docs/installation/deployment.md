# Appliance vs Cluster

Assemblyline support two deployments type; single host which we call "Appliance" and multi-host or "Cluster". They are the same in terms of analysis capabilities however a cluster deployment can be scaled to scan multiple millions of files per day and offer redundancy and fail over. If you are deploying in the cloud a cluster will be easier to deploy (using cloud kubenetes offerings), however in a lab or to support a incident response team any powerful computer will be able to process thoulsand of files a day.

Keep in mind that you will need extra host for running external resources such as anti-virus products, or sandboxes (such as [Cuckoo Sandbox](https://cuckoosandbox.org/)). Theses complementary products are not mandatory but will greatly complement the static analysis and file extraction performed by Assemblyline.

![Deployment types](./images/dep_types.png)

See [Appliance](../appliance) or [Cluster](../cluster) installation instructions.