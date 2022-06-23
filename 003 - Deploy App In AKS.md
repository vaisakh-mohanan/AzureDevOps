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
   
## 4. If the site is not loading follow the steps for troubleshoot the issue
   
   ### 1. describe the pod to see the issue
   
      kubectl describe pod <podname> -n <namespace>
      Ex: kubectl describe pod angular-app
   
   ### 2. If it's unauthorise error refer : https://docs.microsoft.com/en-us/troubleshoot/azure/azure-kubernetes/cannot-pull-image-from-acr-to-aks-cluster
   
   ### 3. Quick steps:
         #### 3.1 To check the container registry's health, run the following command:

               az acr check-health --name <myregistry> --ignore-errors --yes

         #### 3.2 To validate whether the container registry is accessible from the AKS cluster
   
               az aks check-acr --resource-group <MyResourceGroup> --name <MyManagedCluster> --acr <myacr>.azurecr.io
             
               Ex: az aks check-acr --resource-group aztech-rg --name awpAKSCluster --acr awpindacr.azurecr.io
               
         #### 3.3 Cause 1: 401 Unauthorized error
               Check wheather the AKS Cluster has suffient role/Permission to access the Azure container Registry
               Check if the AcrPull role assignment is created:
               az role assignment list --scope     /subscriptions/{subscriptionID}/resourceGroups/{resourcegroupname}/providers/Microsoft.ContainerRegistry/registries/{acrname} -o table
               
               Ex: az role assignment list --scope /subscriptions/f0a4a7a3-0c47-4c17-9b9c-086240960f2b/resourceGroups/aztech-rg/providers/Microsoft.ContainerRegistry/registries/awpindacr -o table
               
       
         #### 3.4 If the AcrPull role assignment isn't created, create it by
         
               az aks update -n {AKSClusterName} -g {resourcegroupname} --attach-acr {acrname}
         
               Ex : az aks update -n awpAKSCluster -g aztech-rg--attach-acr awpindacr
               
               
               
             
   
   
