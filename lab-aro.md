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

## Intro

### Prerequisites

- [Request an increase for vCPU quota.](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/per-vm-quota-requests) (Minimum of 40 vCPU for ARO).
- Create a Red Hat account at [https://console.redhat.com](https://console.redhat.com).
- [Azure CLI](https://boxboat.github.io/k8s-on-azure-wkshp/lab-prerequisites/#3-optional-using-the-azure-cli) version 2.6.0 or later
- Obtain a Red Hat pull secret to enable your cluster to access exclusive Red Hat content and tools. [Download your pull secret from here.](https://cloud.redhat.com/openshift/install/azure/aro-provisioned)

### Resources

- [Microsoft Docs](https://docs.microsoft.com/azure/openshift)
- [Red Hat Docs](https://docs.openshift.com/aro/4/welcome/index.html)


## Creating an ARO cluster

### Setup

In your bash console, set the following variables:

```bash
LOCATION=eastus
RESOURCEGROUP=aro-rg
CLUSTER=cluster
```

Verify your vCPU quota is sufficient to deploy ARO (at least 40 vCPUs):

```bash
az vm list-usage -l $LOCATION \
--query "[?contains(name.value, 'standardDSv3Family')]" \
-o table
```

Register necessary resource providers in your Azure Subscription:

```bash
az provider register -n Microsoft.RedHatOpenShift --wait
az provider register -n Microsoft.Compute --wait
az provider register -n Microsoft.Storage --wait
az provider register -n Microsoft.Authorization --wait
```

### Deploy the foundation
Create a resource group:

```bash
az group create \
  --name $RESOURCEGROUP \
  --location $LOCATION
```

Create a virtual network:

```bash
az network vnet create \
   --resource-group $RESOURCEGROUP \
   --name aro-vnet \
   --address-prefixes 10.0.0.0/22
```

Create an empty subnet for the __master nodes__:

```bash
az network vnet subnet create \
  --resource-group $RESOURCEGROUP \
  --vnet-name aro-vnet \
  --name master-subnet \
  --address-prefixes 10.0.0.0/23 \
  --service-endpoints Microsoft.ContainerRegistry
```

Create an empty subnet for the __worker nodes__:

```bash
az network vnet subnet create \
  --resource-group $RESOURCEGROUP \
  --vnet-name aro-vnet \
  --name worker-subnet \
  --address-prefixes 10.0.2.0/23 \
  --service-endpoints Microsoft.ContainerRegistry
```

[Disable subnet private endpoint policies](https://docs.microsoft.com/en-us/azure/private-link/disable-private-link-service-network-policy) on the master subnet:

```bash
az network vnet subnet update \
  --name master-subnet \
  --resource-group $RESOURCEGROUP \
  --vnet-name aro-vnet \
  --disable-private-link-service-network-policies true
```

### Create the cluster

Run the following command to create your cluster. Ensure that `@pull-secret.txt` is the actual path to your pull secret file that you downloaded during the prerequisites.

```bash
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  --pull-secret @pull-secret.txt
```

Once you run `az aro create`, it usually takes around 30 minutes for the cluster to be fully operational. Once it's ready, continue to the next section to connect to your cluster.

## Connecting to an ARO cluster

### Web Console

Obtain the web console URL for your cluster:

```bash
az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

Open the link that is returned in your browser.

Return to your terminal and obtain the credentials to your cluster:

```bash
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

The credentials you just received can be used to log into the web console. Return to your browser and give it a try!

### Command Line

Install the appropriate version of the [OpenShift CLI](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/) tool for your machine.

Retrieve the address of you OpenShift API:

```bash
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

Obtain the credentials to your cluster:

```bash
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

Log in to your cluster:

```bash
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## Connecting an ARO cluster to the Hybrid Cloud Console

Obtain the pull secrets file from your cluster. The following command saves this as a JSON file.

```bash
oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > pull-secrets.json
```

Open `pull-secrets.txt` that you previously obtained from console.redhat.com and copy the `cloud.openshift.com` object from that file into `pull-secrets.json. Once done, validate your json with the following command:

```bash
cat pull-secret.json | jq
```

And lastly, push that json file back to your cluster with the following command:

```bash
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=./pull-secret.json
```

After a few moments, you can visit the [Red Had Hybrid Cloud Console](https://console.redhat.com/openshift) and view your cluster and all the telemetry being sent there.

## Deploy an application to ARO

```
oc run party-clippy --generator=run-pod/v1 --image=r.j3ss.co/party-clippy
```

Inspect the YAML
``` shell
oc get pod/party-clippy -o yaml
```

To see clippy, we must espose the pod to the internet. Let's create a Kubernetes _service_.

``` shell
oc expose pod/party-clippy --port 80 --target-port 8080 --type LoadBalancer
```

Now, view the YAML. 
```shell
oc get service/party-clippy
```

You'll notice an IP Address under `status.IPAddress`. 

Open up your browser, type in the IP Address.

Hello Clippy!

```
 _________________________________
/ It looks like you're building a \
\ microservice.                   /
 ---------------------------------
 \
  \
     __
    /  \
    |  |
    @  @
    |  |
    || |  /
    || ||
    |\_/|
    \___/
```

## Clean-Up

Once you are ready to dispose of your ARO cluster, you can run the following command:

```bash
az aro delete --resource-group $RESOURCEGROUP --name $CLUSTER
```

Upon completion, all resources belonging to your ARO cluster will be deleted.
