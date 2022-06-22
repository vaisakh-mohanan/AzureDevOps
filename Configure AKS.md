# Deploy an Azure Kubernetes Service (AKS) cluster
Reference  : https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster?tabs=azure-cli

## 1.Create a Kubernetes cluster
  
  az aks create `
    --resource-group {RESOURCEGROUP_NAME} `
    --name {AKS_CLUSTER_NAME} `
    --node-count 2 `
    --generate-ssh-keys `
    --attach-acr {ACR_NAME}
  
  Ex:
  
  az aks create `
    --resource-group aztech-rg `
    --name awpAKSCluster `
    --node-count 2 `
    --generate-ssh-keys `
    --attach-acr awpindacr
    
    Note down SSH Details : SSH key files 'C:\Users\aztech\.ssh\id_rsa' and 'C:\Users\aztech\.ssh\id_rsa.pub'
    
## 2. Create a service priciple to allow AKS serviec to Access Container registry
  follow : https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-service-principal
  
  $ACR_NAME='awpindacr'
  $SERVICE_PRINCIPAL_NAME='awpacrsp'
  $ACR_REGISTRY_ID = $(az acr show --name $ACR_NAME --query "id" --output tsv)
  #Create the service principal with rights scoped to the registry.
  #Default permissions are for docker pull access. Modify the '--role'
  #argument value as desired:
  #acrpull:     pull only
  #acrpush:     push and pull
  #owner:       push, pull, and assign roles
  $PASSWORD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role acrpull --query "password" --output tsv)
  $USER_NAME=$(az ad sp list --display-name $SERVICE_PRINCIPAL_NAME --query "[].appId" --output tsv)
  echo "Service principal ID: $USER_NAME"
  echo "Service principal password: $PASSWORD" 
  
  Note : Might be some issues whie creating Service principle above if you are not a account owner or lower permissions
  In that Case use Managed Identity
  refer : https://docs.microsoft.com/en-us/azure/aks/use-managed-identity
  
  ### Create AKS with managed Identity
  az aks create -g {RESOURCEGROUP_NAME} -n {AKS_CLUSTER_NAME} --enable-managed-identity
  
  Ex: az aks create -g aztech-rg -n awpAKSCluster --enable-managed-identity
  
  ### Get Credentials of AKS 
  az aks get-credentials --resource-group {RESOURCEGROUP_NAME} --name {AKS_CLUSTER_NAME}
  
  Ex : az aks get-credentials --resource-group aztech-rg --name awpAKSCluster
  
  O/P : find config in C:\Users\aztech\.kube\config
  
## 3. Install the Kubernetes CLI

  az aks install-cli
  
## 4. Connect to cluster using kubectl

  ### Gets credentials for the AKS cluster 
    az aks get-credentials --resource-group {RESOURCEGROUP_NAME} --name {AKS_CLUSTER_NAME}
  
  ### To verify the connection to your cluster
    kubectl get nodes
  
  
  



