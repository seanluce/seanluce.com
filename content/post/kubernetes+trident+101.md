---
title: "Azure NetApp Files + Trident = Dynamic and Persistent Storage for Kubernetes"
date: 2020-11-16T03:46:47-04:00
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
draft: false
#thumbnailImage: //example.com/image.jpg
---
<img src="/img/anftridentk8s.png" height="300" />

My eleven year old daughter recently asked me 'the question'. You know the one...

### "Hey dadda, what's Kubernetes?"

How do you explain kubernetes to an eleven year old?! Check out [this video](https://www.linkedin.com/posts/phoebegoh_kubernetes-kubecon-netapp-activity-6700197940151017472-GBn-) to learn how. Brilliant.

I am not a Kubernetes expert. I am learning, probably just like you are. Creating resources, tearing down resources, building them again, kubectl-ing, scratching my head, scratching my head some more, rinse and repeat. I can understand how powerful it is. I can also understand how complex it is.  

As an engineer, I am constantly trying to distill complex things down to their simplest form. I figure out how to do something by reading documentation, blogs, dissecting code, and a humorous amount of trial and error. If I don't distill this information and document it, I'll have to do it all over again in a few months as my memory is just awful. I do enjoy this process, and I hope these types of posts can help you get to where you are going a tiny bit faster.

### If you want to deploy or demonstrate dymamically provisioned, persistent storage without being a Kubernetes expert, this is the post for you.

Let's start with some basic definitions:

  1. [Kubernetes](https://kubernetes.io/); container orchestration system
  2. [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/?ef_id=c1df9981644913634f44c626217fb44d:G:s&OCID=AID2100365_SEM_c1df9981644913634f44c626217fb44d:G:s&msclkid=c1df9981644913634f44c626217fb44d) (AKS); a fully managed Kubernetes cluster service provided by Microsoft
  3. [NetApp Trident](https://netapp-trident.readthedocs.io/en/stable-v20.10/); dynamic, persistent, storage orchestrator for Kubernetes (or Docker)
  4. [Azure NetApp Files](https://azure.microsoft.com/en-us/services/netapp/); lightning fast, enterprise grade, file storage service (NFS/SMB) provided by Microsoft

At the end of this tutorial you will have deployed AKS, installed NetApp Trident, configured a Trident backend to dynamically provision Azure NetApp Files volumes, and have a running nginx deployment being served by Azure NetApp Files storage.

Before we dive in... You will need a Linux operating system to interact with Trident (tridentctl). I am using an Ubuntu VM running in Azure. You could use the Windows Subsystem for Linux or any Linux distro on the supported host operating systems list found [here](https://netapp-trident.readthedocs.io/en/stable-v20.10/support/requirements.html).

Ok, let's dive in!

#### Deploy your Kubernetes cluster using the Azure Kubernetes Service (AKS)

1. From within the Azure portal, navigate to 'Kubernetes services' and click the '+Add' button at the top, choose 'Add Kubernetes cluster'.
![AKS Add Cluster](/img/aks101_addcluster.png)
1. You can accept the default settings for everything. You will need to provide the resource group and give your cluster a name. Feel free to reduce the number of nodes and node size to save yourself some money. A single node is enough for this demo. AKS will create all of the required Azure networking components for you.

#### Create a delegated subnet for Azure NetApp Files

1. Navigate to 'Virtual networks' within the Azure portal. Find your newly created virtual network. It should have a prefix similiar to 'aks-vnet'. Click on the name of the VNet.
![AKS VNet](/img/aks101_newvnet.png)
2. Click on 'Subnets' and select '+Subnet' from the top toolbar.
![ANF New Subnet](/img/aks101_addsubnet.png)
3. Give your subnet a name like 'ANF.sn' and under the 'Subnet delegation' heading, select 'Microsoft.Netapp/volumes'. Do not change anything else. Click 'OK'.
![ANF Subnet Detail](/img/aks101_subnetdetail.png)
<!--more-->

#### Create a new NetApp account within the Azure NetApp Files Service

1. Navigate to the Azure NetApp Files service, click on the '+Add' button. 
![ANF Add Account](/img/aks101_addanfaccount.png)
2. Give your account a name, choose a resource group. IMPORTANT: For the location, choose the same region as your AKS cluster. Click the 'Create' button.
![ANF New Account](/img/aks101_createanfaccount.png)

#### Create a Azure NetApp Files Capacity Pool

1. Navigate to your NetApp account, click on 'Capacity pools', click '+Add pool', give it a name, choose 'Standard' for the service level, leave the QoS type as Auto. Click 'Create'.
![AKS Add Pool](/img/aks101_newpool.png)

#### Install 'kubectl' and the Azure CLI on your Ubuntu VM (or WSL)

   1. [Install 'kubectl'](https://kubernetes.io/docs/tasks/tools/install-kubectl/):

      ```sh
      sudo snap install kubectl --classic
      ```

   2. [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest):

      ```sh
      curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
      ```

   3. Login to your Azure account using Azure CLI

      ```sh
      az login
      ```

#### Connect to your Kubernetes cluster

   1. From within the Azure portal, navigate to your Kubernetes cluster and click the 'Connect' button, execute the 'az account set...' and 'az aks get-credentials...' commands from your Linux terminal.
   ![Connect AKS](/img/aks101_connectaks.png)
   2. Verify you are connected to your Kubernetes cluster

      ```sh
      kubectl get deployments --all-namespaces=true
      ```

      It should look something like this:
      ![Verify AKS](/img/aks101_verifyconnect.png)

#### Install Trident using the 'Trident Operator' method

I encourage you to go read the official documentation [here](https://netapp-trident.readthedocs.io/en/stable-v20.10/kubernetes/deploying/operator-deploy.html#deploying-with-the-trident-operator). I have summarized the commands below.

1. Download the Trident installer bundle

   ```sh
   wget https://github.com/NetApp/trident/releases/download/v20.10.0/trident-installer-20.10.0.tar.gz
   ```

2. Extract the bundle

   ```sh
   tar -xf trident-installer-20.10.0.tar.gz
   ```

3. Change directory to the trident-installer

   ```sh
   cd trident-installer
   ```

4. Deploy the TridentProvisioner Custom Resource Definition (CRD)

   ```sh
   kubectl create -f deploy/crds/trident.netapp.io_tridentprovisioners_crd_post1.16.yaml
   ```

   It should look like this so far:
   ![Install Trident CRD](/img/aks101_installtrident.png)

5. Create the Trident namespace

   ```sh
   kubectl apply -f deploy/namespace.yaml
   ```

   ![Install Trident Operator](/img/aks101_installoperator.png)

6. Deploy the Trident operator to the default 'trident' namespace

   ```sh
   kubectl apply -f deploy/bundle.yaml
   ```

   ![Install Trident CRD](/img/aks101_installtridentbundle.png)

7. Confirm the Trident operator is installed and running

   ```sh
   kubectl get deployment -n trident
   kubectl get pod -n trident
   ```

   ![Confirm Trident Operator Install](/img/aks101_confirmtridentinstall.png)

8. Create the TridentProvisioner Custom Resource (CR) to install Trident

   ```sh
   kubectl create -f deploy/crds/tridentprovisioner_cr.yaml
   ```

   ![Install Trident Custom Resource](/img/aks101_tridentprovisioner.png)

9. Finally, copy 'tridentctl' to a directory in your $PATH (i.e., '/usr/local/bin')

   ```sh
   sudo cp ./tridentctl /usr/local/bin
   ```

### Clone my 'ANF_Trident_AKS' GitHub Repository

This repo has some boiler plate code that we will modify to demostrate Trident in your AKS environment.

1. From your Linux workstation, go back to your home directory

   ```sh
   cd ~
   ```

2. Clone the 'ANF_Trident_AKS' Repository and change directory to 'ANF_Trident_AKS'

   ```sh
   git clone https://github.com/seanluce/ANF_Trident_AKS.git
   cd ANF_Trident_AKS
   ```

### Create an Azure Service Principal

1. The service principal is how Trident communicates with Azure to manipulate your Azure NetApp Files resources.

```sh
az ad sp create-for-rbac --name "http://netapptrident"
```

The output should look like this:

![Create SPN](/img/aks101_serviceprincipal.png)

### Create your Trident backend for Azure NetApp Files

1. Using your preferred text editor, complete the following fields inside the 'anf_backend.json' file:

   * subscriptionID: your Azure Subscription ID
   * tenantID: your Azure Tenant ID (from the output of 'az ad sp' in the previous step)
   * clientID: your appID (from the output of 'az ad sp' in the previous step)
   * clientSecret: your 'password' (from the output of 'az ad sp' in the previous step)
   * location: the location of your Azure NetApp Files capacity pool
   * serviceLevel: the service level of your Azure NetApp Files capacity pool
   * virtualNetwork: the virtual network that was created by AKS
   * subnet: the Azure NetApp Files delegated subnet

   The file should look similiar to this:

   ```json
   {
      "version": 1,
      "storageDriverName": "azure-netapp-files",
      "subscriptionID": "11111111-1111-1111-1111-111111111111",
      "tenantID": "22222222-2222-2222-2222-222222222222",
      "clientID": "33333333-3333-3333-3333-333333333333",
      "clientSecret": "44444444-4444-4444-4444-444444444444",
      "location": "eastus2",
      "serviceLevel": "Standard",
      "virtualNetwork": "aks-vnet-22885919",
      "subnet": "ANF.sn",
      "nfsMountOptions": "vers=3,proto=tcp,timeo=600",
      "limitVolumeSize": "500Gi",
      "defaults": {
         "exportRule": "0.0.0.0/0",
         "size": "200Gi"
      }
   }
   ```

2. Instruct Trident to create the ANF backend in the 'trident' namespace

   ```sh
   tridentctl -n trident create backend -f anf_backend.json
   ```

   ![Create Backend](/img/aks101_createbackend.png)

### Create a Kubernetes storage class

1. Instruct Kubernetes to create a [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) that will reference our Trident backend

   ```sh
   kubectl apply -f anf_sc.yaml
   ```

### Create a Kubernetes persistent volume claim (PVC)

1. Instruct Kubernetes to create a persistent volume claim

   ```sh
   kubectl apply -f anf_pvc.yaml
   ```

2. Verify that Trident has dynamically provisioned an Azure NetApp Files volume. From within the Azure portal you should see that a new Azure NetApp Files volume has been created.

![ANF New Volume](/img/aks101_newvolume.png)

### Create a Kubernetes deployment that references our persistent volume claim (PVC)

1. Instruct Kubernetes to deployment a simple nginx web server

   ```sh
   kubectl apply -f nginx_deployment.yaml
   ```

2. Instruct Kubernetes to give our nginx deployment a public IP address

   ```sh
   kubectl expose deployment nginx-anf-trident --port=80 --type=LoadBalancer
   ```

   ![Expose nginx](/img/aks101_exposenginx.png)

   Notice that only after you 'kubectl expose' does our deployment get a public IP address as shown above in the 'EXTERNAL-IP' column.

3. Get your nginx pod name (it should start with 'nginx-anf-trident')

   ```sh
   kubectl get pods
   ```

   ![Get Pod Name](/img/aks101_getpodname.png)

4. Assign your pod name to an environment variable to save some key strokes

   ```sh
   pod=<paste your pod name from step 3 above here>
   ```

5. Set permissions on our Azure NetApp Files volume. Our volume is nounted via NFSv3 to our pod's '/usr/share/nginx/html' directory. The permissions on this folder need to be '755'.

   ```sh
   kubectl exec -it $pod -- chmod 755 /usr/share/nginx/html
   ```

6. Copy our custom 'index.html' file to your pod's '/usr/share/nginx/html' directory.

   ```sh
   kubectl cp ./index.html $pod:/usr/share/nginx/html/
   ```

7. Point your web browser to your deployment's 'EXTERNAL-IP' from step 2. You should be greeted with our custom index.html.

![Welcome to ANF](/img/aks101_welcometoanf.png)

### Destroy our Kubernetes deployment and re-deploy

1. Delete your Kubernetes service (external IP address)

   ```sh
   kubectl delete svc nginx-anf-trident
   ```

2. Delete your Kubernetes nginx deployment

   ```sh
   kubectl delete -f nginx_deployment.yaml
   ```

3. Instruct Kubernetes to deploy your nginx web server again

   ```sh
   kubectl apply -f nginx_deployment.yaml
   ```

4. Instruct Kubernetes to expose your deployment, giving it a public IP address.

   ```sh
   kubectl expose deployment nginx-anf-trident --port=80 --type=LoadBalancer
   ```

5. Get your new public IP address

   ```sh
   kubectl get svc
   ```

6. Point your browser to the public IP address and notice that your custom index.html has persisted! Magic.

### Cleaning Up

1. Delete your Kubernetes service (external IP address)

   ```sh
   kubectl delete svc nginx-anf-trident
   ```

2. Delete your Kubernetes nginx deployment

   ```sh
   kubectl delete -f nginx_deployment.yaml
   ```

3. Delete your Azure NetApp Files persistent volume claim (PVC)

   WARNING: This command will instruct Trident to delete the Azure NetApp Files volume.

   ```sh
   kubectl delete -f anf_pvc.yaml
   ```

4. Go back to the Azure portal and delete your Azure NetApp Files capacity pool.
