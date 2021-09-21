---
# Page settings
layout: default
keywords:
comments: false

# Hero section
title: Lab - Intro to Azure RedHat OpenShift (ARO)
description: Let's find out the differences with Azure Kubernetes Service (AKS)

# Author box
author:
    title: About Author
    title_url: '#'
    external_url: true
    description: Justin is a solutions architect at BoxBoat. He was been working with Azure for many years. Sometimes, he goes by Jimmy.

# Micro navigation
micro_nav: true

# Page navigation
page_nav:
    prev:
        content: Lab - Intro to Azure Kubernetes Service (AKS)
        url: '/lab-aks'
---

## Prerequisites

- [Request an increase for vCPU quota.](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/per-vm-quota-requests) (Minimum of 40 vCPU for ARO).
- Create a Red Hat account at [https://console.redhat.com](https://console.redhat.com).
- [Azure CLI](https://boxboat.github.io/k8s-on-azure-wkshp/lab-prerequisites/#3-optional-using-the-azure-cli) version 2.6.0 or later

## Resources

- [Microsoft Docs](https://docs.microsoft.com/azure/openshift)
- [Red Hat Docs](https://docs.openshift.com/aro/4/welcome/index.html)

## Goals

- Create an ARO Cluster.
- Connect to cluster via command line.
- Explore cluster via web console.
- Connect cluster to Red Hat Hybrid Cloud Console.
- Deploy a workload to cluster.

## Creating an ARO cluster

### Setup

In your bash console, set the following variables:

```bash
LOCATION=eastus
RESOURCEGROUP=aro-rg
CLUSTER=cluster
```

Verify your vCPU quota is sufficient to deploy ARO (at least 40 vCPUs):

```
az vm list-usage -l $LOCATION \
--query "[?contains(name.value, 'standardDSv3Family')]" \
-o table
```

Register necessary resource providers in your Azure Subscription:
```
az provider register -n Microsoft.RedHatOpenShift --wait
az provider register -n Microsoft.Compute --wait
az provider register -n Microsoft.Storage --wait
az provider register -n Microsoft.Authorization --wait
```

## Connecting to an ARO cluster

## Exploring an ARO cluster via Web Console

## Connecting an ARO cluster to the Hybrid Cloud Console

## Deploy a workload to ARO

## Clean-Up
