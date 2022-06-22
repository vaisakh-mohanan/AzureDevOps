
# Deploying a Angular App to Azure Container Registry From Git
============================================================
	Reference : https://medium.com/swlh/deploy-an-angular-app-to-azure-955f0c750686
  For Implementation refer : https://github.com/vaisakh-mohanan/Angular-ACR-Demo

## 1. Add Dockerfile with below Script Inside Near to Package.json file
	
	# Stage 1: Build
	FROM node:18-alpine AS build
	WORKDIR /usr/src/app
	COPY package.json ./
	RUN npm install
	COPY . .
	RUN npm run build

	# Stage 2: Run 
	FROM nginx:1.21.6-alpine
	# Copy compiled files from previous build stage
	COPY --from=build /usr/src/app/dist/DIMS /usr/share/nginx/html
	
## 2. Add .dockerignore file with below content
	
	node_modules
	.git
	.gitignore
	
## 3. Install AzureCli and Login or use Cloud Cli

## 4. Create a Resource Group
	
	az group create --name {ResourceGroupName} --location eastus
	
## 5. Create a Container Registry To hold docker Images

	az acr create --resource-group {ResourceGroupName} `
	--name {ContainerRegistryName} `
	--sku Basic `
	--subscription {SubcriptionName} 
	
	Example:
	
	az acr create --resource-group aztech-rg `
	--name awpindacr `
	--sku Basic `
	--subscription aztecld-westeurope-dev 
	
## 6.Now, login to your new Container Registry so that we can extract some container information, such as its login server name

	### 6.1	
		az acr login --name {ContainerRegistryName}
		
		Ex: az acr login --name awpindacr
	
	### 6.2
		az acr show --name {ContainerRegistryName} `
		--query loginServer `
		--output table `
		--subscription {SubcriptionName}
		
		Ex: 
		
		az acr show --name awpindacr `
		--query loginServer `
		--output table `
		--subscription aztecld-westeurope-dev
		
		Output : awpindacr.azurecr.io
		
## 7. Re-deploy your App from GitHub : your app re-build and deploy whenever you make changes to a GitHub repository

	For that Use Azure Container Tasks

## 8.	Set Variables For Container Registry Tasks
	
	$ACR_NAME={ContainerRegistryName} # The name of your Azure container registry
	$GIT_USER={yourusername} # Your GitHub user account name
	$GIT_PAT=e04c06c3b571babe4e712ba0347a64119a61b61a # The PAT you generated in the previous section
	
	example :
	
	$ACR_NAME='awpindacr'       # The name of your Azure container registry`
	$GIT_USER='vaisakh-mohanan'      # Your GitHub user account name`
	$GIT_PAT='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' # The PAT you generated in the previous section
	
## 9.	Create the Task
	
	az acr task create `
	--registry $ACR_NAME `
	--name {ACRTaskName} `
	--image {login server name of the container registry}/{ImageName}:{Tag} `
	--context https://github.com/$GIT_USER/Angular-Tour-of-Heroes.git#{branchName} `
	--file Dockerfile `
	--git-access-token $GIT_PAT
	
	Example1: (Not works due to github access issue)
	
	az acr task create `
	--registry $ACR_NAME `
	--name awpgitpull `
	--image awpindacr.azurecr.io/dimsui:prod `
	--context https://github.developer.allianz.io/AllianzPartnersApplicationDevelopment/$GIT_USER/DIMS_UI.git#master `
	--file Dockerfile `
	--git-access-token $GIT_PAT
	
	Example 2:
	
	$GIT_PAT='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'
	
	az acr task create `
	--registry $ACR_NAME `
	--name awpgitpull `
	--image awpindacr.azurecr.io/angularsample:prod `
	--context https://github.com/vaisakh-mohanan/Angular-ACR-Demo.git#master `
	--file Dockerfile `
	--git-access-token $GIT_PAT
	
	Example 3: with argumets
	az acr task create `
    --registry $ACR_NAME `
    --name awpgitpullnew `
    --image angularsample:$(date +%m%d%Y) `
    --arg REGISTRY_NAME=$ACR_NAME.azurecr.io `
    --context https://github.com/vaisakh-mohanan/Angular-ACR-Demo.git `
    --file Dockerfile `
    --branch azure `
    --git-access-token $GIT_PAT
	
	
## 10. Create Task from yaml file config

	### 10.1- Create a yaml file with build config 
	 Ex: 
		version: v1.1.0
		steps:
		  - build: -t $Registry/angular-app:$ID -t $Registry/angular-app:latest -f Dockerfile .
		  - push:
			- $Registry/angular-app:$ID
			- $Registry/angular-app:latest
			
	### 10.2 - Create ACR Task with yaml file reference
	
		Ex:
		  az acr task create --name awpgitpullyaml `
		  --registry $ACR_NAME `
		  --commit-trigger-enabled true `
		  --pull-request-trigger-enabled false `
		  --base-image-trigger-enabled false `
		  --context https://github.com/vaisakh-mohanan/Angular-ACR-Demo.git#master `
		  --assign-identity $MSI_ID `
		  --file build.yaml `
		  --git-access-token $GIT_PAT
      
      
# Deploy the Web App from your Container Image

## 1. Create an App Service Plan for your web app
      This will determine the size of the virtual machine that will run your docker image. Here I have selected a sku of F1 — free tier. Create the app service plan     from within the portal or by running:
      
      az appservice plan create `
      --resource-group {RESOURCEGROUPNAME} `
      --name {APPSERVICENAME} `
      --location eastus `
      --is-linux `
      --sku F1
      
      Ex:
       az appservice plan create `
      --resource-group aztech-rg `
      --name angularAppDemoPlan `
      --location eastus `
      --is-linux `
      --sku F1
      
## 2. Create the Azure web app from the docker container in the Container Registry

  az webapp create --resource-group {RESOURCEGROUPNAME} `
  --plan {APPSERVICEPLANNAME} `
  --name {WEBAPPNAME} `
  --deployment-container-image-name {REGISTRYNAME/IMAGENAME:IMGTAG} `
  --docker-registry-server-password {PASSWORD}
  
  Ex:
  
   az webapp create --resource-group aztech-rg `
  --plan angularAppDemoPlan `
  --name angularAppDemo `
  --deployment-container-image-name awpindacr.azurecr.io/angular-app:latest `
  --docker-registry-server-password 'HVZu1qbuBNpBw4vnqDyMYb2=rOKVqL0N'
  --subscription aztecld-westeurope-dev
  
  Note : To vie Password : az acr credential show -n {REGISTRY NAME} --query passwords[0].value
