---
title: "kubernetes: Dynamic and Persistent with Trident and Azure NetApp Files"
date: 2020-08-17T15:46:47-04:00
categories:
- azure netapp files
- kubernetes
tags:
- kubernetes
- azure
- anf
- trident
keywords:
- kubernetes
- azure
- anf
- trident
draft: true
#thumbnailImage: //example.com/image.jpg
---
I am not a Kubernetes expert. I am learning, probably just like you are. Creating resources, tearing down resources, building them again, scratching my head, scratching my head some more, rinse and repeat. I can understand how powerful it is. I can also understand how complex it is. My eleven year old daughter recently asked me what kubernetes was. How do you explain kubernetes to an eleven year old?! Check out [this video](https://www.linkedin.com/posts/phoebegoh_kubernetes-kubecon-netapp-activity-6700197940151017472-GBn-) to learn how. Brilliant.

As an engineer, I am constantly trying to distill complex things down to their simpliest form. I figure out how to do something by reading documentation, blogs, dissecting code, and a humorous amount of trial and error. If I don't distill this information and document it, I'll have to do it all over again in a few months as my memory is just awful. I do enjoy this process, and I hope these types of posts can help you get to where you are going a tiny bit faster.

### If you want to deploy or demonstrate dymamically provisioned, persistent storage without being a kubernetes expert, this is the post for you.

Let's start with some basic definitions:

  1. [Kubernetes](https://kubernetes.io/); container orchestration system
  2. [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/?ef_id=c1df9981644913634f44c626217fb44d:G:s&OCID=AID2100365_SEM_c1df9981644913634f44c626217fb44d:G:s&msclkid=c1df9981644913634f44c626217fb44d) (AKS); a fully managed Kubernetes cluster provided by Microsoft Azure
  3. [NetApp Trident](https://azure.microsoft.com/en-us/services/kubernetes-service/?ef_id=c1df9981644913634f44c626217fb44d:G:s&OCID=AID2100365_SEM_c1df9981644913634f44c626217fb44d:G:s&msclkid=c1df9981644913634f44c626217fb44d); dynamic persistent storage orchestrator for Kubernetes (or Docker)
  4. [Azure NetApp Files](https://azure.microsoft.com/en-us/services/netapp/); lightning fast, enterprise grade, file storage service (NFS/SMB) provided by Microsoft Azure (for you PVCs)

Before we dive in... You will need Linux to run tridentctl. I am using an Ubuntu VM running in Azure. You could use the Windows Subsystem for Linux or any Linux distro on the supported host operating systems list found [here](https://netapp-trident.readthedocs.io/en/stable-v20.10/support/requirements.html).

1. Deploy your kubernetes cluster using the Azure Kubernetes Service (AKS)
   1. From within the Azure portal, navigate to 'Kubernetes services' and click the '+Add' button at the top, choose 'Add Kubernetes cluster'.
   aks101_addcluster.png
   2. You can accept the default settings for everything. Feel free to reduce the number of nodes and node size to save some money.

2. Create and delegate a subnet to Azure NetApp Files
   1. Find your newly created virtual network, click on 'Subnets' and select '+Subnet' from the top toolbar. Give it a name and select 'Microsoft.Netapp/volumes' under the 'Subnet delegation' heading, click 'OK'.

3. Create a new NetApp account within the Azure NetApp Files Service
   1. Navigate to the Azure NetApp Files service, click on the '+Add' button, give it a name, choose the same location as your kubernetes cluster.

4. Create a Azure NetApp Files Capacity Pool
   1. Navigate to your NetApp account, click on 'Capacity pools', click '+Add pool', give it a name, choose 'Standard' for the service level, click 'Create'.

5. SSH to your Linux workstation and install kubectl and the Azure CLI (yoou can skip this step if you are using Cloud Shell) 
   1. Instructions to install kubectl here: https://kubernetes.io/docs/tasks/tools/install-kubectl/
   2. Instruction to install Azure CLI here: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest

6. Connect to your kubernetes cluster
   1. Navigate to your kubernetes cluster and click the 'Connect' button, paste the 'az account set...' and 'az aks get-credentials...' commands in to your Linux terminal.

7. Verify that your kubernetes cluster is running and that you are connected
   1. kubectl get deployments --all-namespaces=true

8. Install Trident using the 'Trident Operator' method
   1. Download the Trident installer bundle, 'wget https://github.com/NetApp/trident/releases/download/v20.07.0/trident-installer-20.07.0.tar.gz'
   2. Extract the bundle, 'tar -xf trident-installer-20.07.0.tar.gz'
   3. cd trident-installer
   4. Deploy the Trident Operator, 'kubectl create -f deploy/crds/trident.netapp.io_tridentprovisioners_crd_post1.16.yaml'


https://discourse.ubuntu.com/t/deploying-netapp-trident-with-charmed-kubernetes/16768
<!--more-->
