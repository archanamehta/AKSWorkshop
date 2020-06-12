# Create a private, highly available container registry

The Fruit Smoothies software development and operations teams made the decision to containerize all newly developed applications. Containerized applications provide the teams with mutual benefits. For example,

The ease of managing hosting environments
The guarantee of continuity in software delivery
The efficient use of server hardware
The portability of applications between environments.
The teams made the decision to store all containers in a central and secure location and the decision made is to use Azure Container Registry.

# In this exercise, you will:

Create a container registry by using the Azure CLI
Build container images by using Azure Container Registry Tasks
Verify container images in Azure Container Registry
Configure an AKS cluster to authenticate to an Azure Container Registry
Create a container registry

Azure Container Registry is a managed Docker registry service based on the open-source Docker Registry 2.0. Container Registry is private and hosted in Azure. You use it to build, store, and manage images for all types of container deployments.

Container images can be pushed and pulled with Container Registry by using the Docker CLI or the Azure CLI. You can use Azure portal integration to visually inspect the container images in the container registry. In distributed environments, the Container Registry geo-replication feature can be used to distribute container images to multiple Azure datacenters for localized distribution.

Azure Container Registry Tasks can also build container images in Azure. Tasks use a standard Dockerfile to create and store a container image in Azure Container Registry without the need for local Docker tooling. With Azure Container Registry Tasks, you can build on-demand or fully automate container image builds by using DevOps processes and tooling.

Let's deploy a container registry for the Fruit Smoothies environment.

  The container registry name must be unique within Azure and contain between 5 and 50 alphanumeric characters. For learning purposes, run this command from Azure Cloud Shell to create a Bash variable that holds a unique name.

    ACR_NAME=acr$RANDOM
    
    az acr create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $ACR_NAME \
    --sku Standard
    
    
# Build the container images by using Azure Container Registry Tasks
The Fruit Smoothies rating app makes use of two container images, one for the front-end website and one for the RESTful API web service. Your development teams use the local Docker tooling to build the container images for the website and API web service. A third container is used to deploy the document database provided by the database publisher and will not be stored the database container in ACR.

You can use Azure Container Registry to build these containers using a standard Dockerfile to provide build instructions. With Azure Container Registry, you can reuse any Dockerfile currently in your environment, which includes multi-staged builds.

# Build the ratings-api image
The ratings API is a Node.js application that's built using Express, a Node.js web framework. The source code  is on GitHub and already includes a Dockerfile , which builds images based on the Node.js Alpine container image.

Here, you'll clone the repository and then build the Docker image using the included Dockerfile. Use the built-in ACR functionality to build and push the container image into your registry by running the az acr build command.


# Clone the repository to your Cloud Shell.
    git clone https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-api.git
    
    cd mslearn-aks-workshop-ratings-api
    
    az acr build \
    --resource-group $RESOURCE_GROUP \
    --registry $ACR_NAME \
    --image ratings-api:v1 .
    
    
# Build the ratings-web image
The ratings front end is a Node.js application that was built by using the Vue JavaScript framework and WebPack to bundle the code. The source code  is on GitHub and already includes a Dockerfile , which builds images based on the Node.js Alpine image.

The steps you follow are the same as before. Clone the repository and then build the docker image using the included Dockerfile using the az acr build command.


    git clone https://github.com/MicrosoftDocs/mslearn-aks-workshop-ratings-web.git
    cd mslearn-aks-workshop-ratings-web
    
    az acr build \
    --resource-group $RESOURCE_GROUP \
    --registry $ACR_NAME \
    --image ratings-web:v1 .
    
    az acr repository list \
    --name $ACR_NAME \
    --output table
    


# Configure the AKS cluster to authenticate to the container registry
We need to set up authentication between your container registry and Kubernetes cluster to allow communication between the services.

Let's integrate the container registry with the existing AKS cluster by supplying valid values for AKS_CLUSTER_NAME and ACR_NAME. You can automatically configure the required service principal authentication between the two resources by running the az aks update command.

Run the following command.
    az aks update \
        --name $AKS_CLUSTER_NAME \
        --resource-group $RESOURCE_GROUP \
        --attach-acr $ACR_NAME
        
# Summary
In this exercise, you created a container registry for the Fruit Smoothies application. You then built and added container images for the ratings-api and ratings-web to the container registry. You then verified the container images, and configured your AKS cluster to authenticate to the container registry.

Next, you'll take the first step to deploy your ratings app. The first component you'll deploy is MongoDB as your document store database, and you'll see how to use the HELM package manager for Kubernetes.

Next **[Exercise - Deploy Mongo ](https://github.com/archanamehta/AKSWorkshop/blob/master/DeployMongo.md)** 

    




