# Deploy Kubernetes with Azure Kubernetes Service

Fruit Smoothies wants to use Kubernetes as their compute platform. The development teams already use containers for application development and deployment, and using an orchestration platform will help them rapidly build, deliver, and scale their application.

To do this, you need to deploy the foundation of your Kubernetes environment.

In this exercise, you will:

Create a new resource group.
Configure cluster networking.
Create an Azure Kubernetes Service cluster.
Connect to the Kubernetes cluster by using kubectl.
Create a Kubernetes namespace.


# Create a new resource group
You'll first need to create a resource group for your resources to deploy into.

    REGION_NAME=eastus
    RESOURCE_GROUP=aksworkshop
    SUBNET_NAME=aks-subnet
    VNET_NAME=aks-vnet

Create a new resource group with the name aksworkshop. Deploy all resources created in these exercises in this resource group. A single resource group makes it easier to clean up the resources after you finish the module.

    az group create \
        --name $RESOURCE_GROUP \
        --location $REGION_NAME
    
  
  
# What is Kubenet networking?
Kubenet networking is the default networking model in Kubernetes. With Kubenet networking, nodes get assigned an IP address from the Azure virtual network subnet. Pods receive an IP address from a logically different address space to the Azure virtual network subnet of the nodes.

Network address translation (NAT) is then configured so that the pods can reach resources on the Azure virtual network. The source IP address of the traffic is translated to the node's primary IP address and then configured on the nodes. Note, that pods receive an IP address that's "hidden" behind the node IP.

# What is Azure Container Networking Interface (CNI) networking?
With Azure Container Networking Interface (CNI), the AKS cluster is connected to existing virtual network resources and configurations. In this networking model, every pod gets an IP address from the subnet and can be accessed directly. These IP addresses must be unique across your network space and calculated in advance.

Some of the features you'll use require you to deploy the AKS cluster by using the Azure Container Networking Interface networking configuration.

For a more detailed comparison, see the Learn more section at the end of this module.

Let's create the virtual network for your AKS cluster. We will use this virtual network and specify the networking model when we deploy the cluster.

First, create a virtual network and subnet. Pods deployed in your cluster will be assigned an IP from this subnet. Run the following command to create the virtual network.

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --location $REGION_NAME \
        --name $VNET_NAME \
        --address-prefixes 10.0.0.0/8 \
        --subnet-name $SUBNET_NAME \
        --subnet-prefix 10.240.0.0/16
    
    SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
    
    
# Create the AKS cluster
With the new virtual network in place, you can go ahead and create your new cluster. There are two values you need to know before running the az aks create command. The first is the version of the latest, non-preview, Kubernetes version available in your selected region, and the second is a unique name for your cluster.

To get the latest, non-preview, Kubernetes version you use the az aks get-versions command. Store the value that returns from the command in a Bash variable named VERSION. Run the command below the retrieve and store the version number.

    VERSION=$(az aks get-versions \
        --location $REGION_NAME \
        --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' \
        --output tsv)
    
 The AKS cluster name must be unique. Run the following command to create a Bash variable that holds a unique name.
      AKS_CLUSTER_NAME=aksworkshop-$RANDOM
    
Run the following az aks create command to create the AKS cluster running the latest Kubernetes version. This command can take a few minutes to complete.

    az aks create \
    --resource-group $RESOURCE_GROUP \
    --name $AKS_CLUSTER_NAME \
    --vm-set-type VirtualMachineScaleSets \
    --load-balancer-sku standard \
    --location $REGION_NAME \
    --kubernetes-version $VERSION \
    --network-plugin azure \
    --vnet-subnet-id $SUBNET_ID \
    --service-cidr 10.2.0.0/24 \
    --dns-service-ip 10.2.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --generate-ssh-keys


# Test cluster connectivity by using kubectl
kubectl is the main Kubernetes command-line client you use to interact with your cluster and is available in Cloud Shell. A cluster context is required to allow kubectl to connect to a cluster. The context contains the cluster's address, a user, and a namespace. Use the az aks get-credentials command to configure your instance of kubectl.

# GET Credentials 
    az aks get-credentials \
        --resource-group $RESOURCE_GROUP \
        --name $AKS_CLUSTER_NAME
    
    kubectl get nodes

# Create a Kubernetes namespace for the application
Fruit Smoothies want to deploy several apps from other teams in the deployed AKS cluster as well. Instead of running multiple clusters, the company wants to use the Kubernetes features that let you logically isolate teams and workloads in the same cluster. The goal is to provide the least number of privileges scoped to the resources each team needs.

What is a namespace?
A namespace in Kubernetes creates a logical isolation boundary. Names of resources must be unique within a namespace but not across namespaces. If you don't specify the namespace when you work with Kubernetes resources, the default namespace is implied.

Let's create a namespace for your ratings application.

List the current namespaces in the cluster.
    kubectl get namespace
    
Use the kubectl create namespace command to create a namespace for the application called ratingsapp.
    kubectl create namespace ratingsapp



  


  
    
 
