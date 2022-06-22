# Run applications in Azure Kubernetes Service (AKS)
Refer: https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-application?tabs=azure-cli

## 1. Update the manifest file ACR login server Details

   az acr list --resource-group {ResourceGroupName} --query "[].{acrLoginServer:{loginServerName}}" --output table
   
   Ex:
   az acr list --resource-group aztech-rg --query "[].{acrLoginServer:awpindacr.azurecr.io}" --output table
   
## 2. Update kubernetives manifest files
  Sample manifests : https://github.com/Azure-Samples/azure-voting-app-redis/blob/master/azure-vote-all-in-one-redis.yaml
  implementation : https://github.com/vaisakh-mohanan/Angular-ACR-Demo/blob/master/kubebuild.yaml

## 3. Deploy the application
