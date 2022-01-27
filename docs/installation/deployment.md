# Choose your deployment type

Assemblyline has two distinctive deployment types:

 * **Appliance**: Single host deployment
 * **Cluster**: Multi-host deployment

!!! warning
    Keep in mind that you will need extra hosts for running external resources such as anti-virus products, or sandboxes (such as [Cuckoo Sandbox](https://cuckoosandbox.org/)). These complementary products are not mandatory but will greatly complement the static analysis and file extraction performed by Assemblyline.

## Features

Both deployments are the same in terms of analysis capabilities however a cluster deployment can be scaled to scan multiple millions of files per day and offer redundancy and failover. If you are deploying in the cloud, a cluster will be easier to deploy (using cloud Kubernetes offerings). However, in a lab or to support an incident response team, any powerful computer with an appliance deployment will be able to process thousands of files a day.

!!! tip
    If you choose to deploy a MicroK8s appliance, you will be able to scaled it up to a small cluster using multiple machines (nodes) if need be.

### Deployment features rundown

|                                           | Appliance (Docker)                     | Appliance (MicroK8s)                   | Cluster                                |
| ----------------------------------------- | -------------------------------------- | -------------------------------------- | -------------------------------------- |
| Support all analysis capabilities         | :material-checkbox-marked-outline: Yes | :material-checkbox-marked-outline: Yes | :material-checkbox-marked-outline: Yes |
| Single host installation                  | :material-checkbox-marked-outline: Yes | :material-checkbox-marked-outline: Yes | :material-checkbox-blank-outline: No   |
| Easy step by step installation            | :material-checkbox-marked-outline: Yes | :material-checkbox-marked-outline: Yes | :material-checkbox-blank-outline: No   |
| Simple to deploy and manage               | :material-checkbox-marked-outline: Yes | :material-checkbox-blank-outline: No   | :material-checkbox-blank-outline: No   |
| Multiple host installation                | :material-checkbox-blank-outline: No   | :material-minus-box-outline: Optional  | :material-checkbox-marked-outline: Yes |
| Auto-scaling of core components           | :material-checkbox-blank-outline: No   | :material-minus-box-outline: Optional  | :material-checkbox-marked-outline: Yes |
| High volume throughput                    | :material-checkbox-blank-outline: No   | :material-checkbox-blank-outline: No   | :material-checkbox-marked-outline: Yes |
| Cloud provider support (AKS, EKS, GKE...) | :material-checkbox-blank-outline: No   | :material-checkbox-blank-outline: No   | :material-checkbox-marked-outline: Yes |
| Redundancy / Failover support             | :material-checkbox-blank-outline: No   | :material-checkbox-blank-outline: No   | :material-checkbox-marked-outline: Yes |

## Installation stack

Both clustered and MicroK8s deployments use a very similar stack in the background which allows them to share the same Helm chart. Only small changes in the values.yml file are required to differentiate them.

As for the Docker compose appliance deployment, it uses a simpler stack which is easier to maintain and reset but offer less features to scale to high capacity.

![Deployment types](./images/dep_types.png)

## Installation instructions

Now that you know the difference between the two types of deployment, you can refer to their respective installation instruction to get you started.

* [Appliance installation (Docker)](../appliance_docker)
* [Appliance installation (Microk8s)](../appliance)
* [Cluster installation](../cluster)

!!! tip
    Consider reading the [configuration](../configuration/config_file/) section of the documentation before jumping into the installation instruction. This will help you understand all the different options you can modify during the installation process.
