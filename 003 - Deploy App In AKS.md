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

   clone the  application that contains the kubernetes manifest file and run following command
   
   Ex: git clone https://github.com/vaisakh-mohanan/Angular-ACR-Demo.git
 
   kubectl apply -f {Kubernetes_Manifest_file.yaml}
   
   Ex: kubectl apply -f kubebuild.yaml
   
   
## 3. Test the application

   kubectl get service {Service_Name} --watch
   
   Ex: kubectl get service angular-app --watch
   
   Note : Initially the EXTERNAL-IP for the azure-vote-front service is shown as pending and When the EXTERNAL-IP address changes from pending to an actual public IP address and test.
