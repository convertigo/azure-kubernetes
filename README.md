# azure-kubernetes
Deploy Convertigo MBaaS  on Azure using AKS (Kubernetes Services)

# Goal
This repo will help you deploy Convertigo on Azure AKS, the Kubernetes service. 

# Steps
1. Create a new AKS service in Azure with the portal. This can be done by :
* Create a Resource->Containers->Kubernetes Service
* Fill in Information for Resource Group, ClusterName, Choose 1.96 As Version
* Leave Default dor DNS name prefix,  Service Principal, Scale Node Size and Scale Node Count
* Click Next:NetWorking. Be Sure to select HTTP "Application Routing"
* Click Next:Monitoring. Leave as default
* Click Next:tags. Leave as default
* Click Review + Create, and Create the cluster. This will take a Few seconds.
2. When ready, Click on the portal's "All resources", you will see a Whole Bunch of resources there.
* Group the list by type and in the "Kubernetes Service" click on the cluster to open the cluster's properties.
* Then click on the "View Kubernetes dashboard" and follow connection instructions. You will have to install Azure CLI to be able to open the dashboard.
3. Convertigo runs as Multi instance Shared Workspace configuration, so it will need an AzureFile shared "StorageAccount" resource.
* Create a new "Storge account" resource: In the portal Click "Create a resource"->Storage->Storage Account
* set the name explicitly to be "c8oworkspace". This is to match the name used by the provided Convertigo Kubernetes YAML deploy file
* Select "ResourceManager"
* Select the existing  Resource group named "MC_<your clustername>-resource-group_....." This is mandatory to have Convertigo Instances to access this StorageAccount.
* Click "Create" your Storage account will be created
  
  







