---
layout: default
title: Cluster (kubernetes)
parent: Deployment types
grand_parent: Installation
nav_order: 2
has_toc: false
---

# Installing an Assemblyline cluster
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Pre-requisites

A kubernetes cluster close to the "vanilla" kubernetes product such as Rancher, AKS (Azure), EKS (Amazon), GKE (Google).

## Cluster Installation

Kubernetes deployment are done by using a helm chart; our official helm chart can be found here:
[https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml](https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml). 

The helm chart is well commented inline to help.

