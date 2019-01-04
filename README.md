# azure-kubernetes
Deploy Convertigo MBaaS on Azure using AKS (Kubernetes Services)

# Goal
This repo will help you deploy Convertigo on Azure AKS, the Kubernetes service. 

# Steps
1. Create a new AKS service in Azure with the portal. This can be done by :

* Create a Resource->Containers->Kubernetes Service
* Fill in Information for Resource Group, ClusterName, Choose last docker Version
* Leave Default dor DNS name prefix,  Service Principal, Scale Node Size and Scale Node Count
* Leave RBAC to no
* Click Next:NetWorking. Be Sure to select HTTP "Application Routing"
* Click Next:Monitoring. Leave as default
* Click Next:tags. Leave as default
* Click Review + Create, and Create the cluster. This will take a Few seconds.

2. When ready, Click on the portal's "All resources", you will see a Whole Bunch of resources there.
* Group the list by type and in the "Kubernetes Service" click on the cluster to open the cluster's properties.
* Then click on the "View Kubernetes dashboard" and follow connection instructions. You will have to install Azure CLI to be able to open the dashboard.

3. Convertigo runs as Multi instance Shared Workspace configuration, so it will need an AzureFile shared "StorageAccount" resource to hold the shared workspace on a persistent disk. We use for that the Azure's SMB 3.0 shareable "StorageAccount" File resource.

* Create a new "Storge account" resource: In the portal Click "Create a resource"->Storage->Storage Account
* set the name explicitly to be "c8oworkspace". This is to match the name used by the provided Convertigo Kubernetes YAML deploy file
* Select "ResourceManager"
* Select the existing  Resource group named "MC_&lt;your clustername&gt;-resource-group_....." This is mandatory to have Convertigo Instances able to access this StorageAccount.
* Click "Create" your Storage account will be created

4. Install additional components in the cluster for TLS and Letsencrypt
 
* We need HELM, install it from   https://github.com/helm/helm/releases
* HELM is a command line utility, we have to intialize it before to run tiller in the cluster: helm init
* Now install Ingress :
	helm install stable/nginx-ingress --namespace kube-system --set controller.replicaCount=2 --set rbac.create=false
* Install the cert-manager :
	helm install stable/cert-manager --namespace kube-system --set ingressShim.defaultIssuerName=letsencrypt-prod --set ingressShim.defaultIssuerKind=ClusterIssuer --set rbac.create=false  --set serviceAccount.create=false
  
5. Deploy the YAML defintion to the kubernetes cluster

* run kubectl apply -f c8o-kubernetes.yaml
* run the Kubernetes dashboard (az aks browse --resource-group &lt;your resource group&gt; --name &lt;your cluster name&gt;)
* Your Browser will open a new page with the dashboard.. 

# Notes:
This YAML files declares a cluster with 3 Convertigo replicas and 1 fullSync CouchDB repository replica. You can change this for your own deployments.

You might have to change the hosts definitions in the following section to match your DNS FDQN (here : demo-c8o.westeurope.cloudapp.azure.com)

```
spec:
  tls:
  - hosts:
    - demo-c8o.westeurope.cloudapp.azure.com
    secretName: example-tls
  rules:
  - host: demo-c8o.westeurope.cloudapp.azure.com
```
# Using a CouchDB Cluster
This sample YAML uses a single instance CouchDB. If you want to use a Multi instance clustered CouchDB, you can install a CouchDB Cluster using a HELM chart. see https://github.com/helm/charts/tree/master/incubator/couchdb

You can use this command to install a 6 instance CouchDB 2.3.0 cluster, each one using an an AzureDisk of 1Gb. The data will be replicated on all these instances so that if we loose a node, the cluster will still work. Also, as the requests to the CouchDB instances are load balanced using a NodePort, we will benefit from Parallel processing for CouchDB requests such as the _changes and the _bulk_get requests.

```
helm install --name cdb-fullsync incubator/couchdb --set clusterSize=6 --set persistentVolume.enabled=true --set persistentVolume.size=1Gi --set image.tag=2.3.0 --set service.type=NodePort
```




  
  







