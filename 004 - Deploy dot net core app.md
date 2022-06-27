
# Deploying a Dot Net Core App to Azure Container Registry From Git
For common steps like creating Azure resources follow the link : https://github.com/vaisakh-mohanan/AzureDevOps/blob/master/001%20-%20Push%20Angular%20Image%20to%20ACR.md

  ## 1. Add Dockerfile with below Script 

    FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build
    WORKDIR /app

    #copy csproj and restore as distinct layers
    COPY *.sln .
    COPY api/*.csproj ./aspnetapp/
    RUN dotnet restore

    #copy everything else and build app
    COPY aspnetapp/. ./aspnetapp/
    WORKDIR /app/aspnetapp
    RUN dotnet publish -c Release -o out

    FROM mcr.microsoft.com/dotnet/core/aspnet:3.0 AS runtime
    WORKDIR /app
    COPY --from=build /app/aspnetapp/out ./
    ENTRYPOINT ["dotnet", "aspnetapp.dll"]
  
  ## 2. Add .dockerignore file with below content
  
    # directories
    **/bin/
    **/obj/
    **/out/

    # files
    Dockerfile*
    **/*.md
    
 ## 3. Create a Container Registry To hold docker Images
 
 ## 4. Now, login to your new Container Registry so that we can extract some container information, such as its login server name
 
 ## 5. Use ACR Taks to re-build and deploy app whenever you make changes to a GitHub repository
 
 ## 6. Create Task from yaml file config
 
  ### 10.1- Create a yaml file with build config
    Ex: 
    version: v1.1.0
    steps:
      - build: -t $Registry/aspnet-app:$ID -t $Registry/aspnet-app:latest -f Dockerfile .
      - push:
      - $Registry/aspnet-app:$ID
      - $Registry/aspnet-app:latest
		
  ### 10.2 - Create ACR Task with yaml file reference
  
    $GIT_PAT='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'

    Ex:
      az acr task create --name awpaspnetgitpullyaml `
      --registry $ACR_NAME `
      --commit-trigger-enabled true `
      --pull-request-trigger-enabled false `
      --base-image-trigger-enabled false `
      --context https://github.com/vaisakh-mohanan/DotNetCore-ACR-Demo.git#master `
      --assign-identity $MSI_ID `
      --file build.yaml `
      --git-access-token $GIT_PAT
 
 
 
 
 
 
 
 
 
 
 
 
 
 
