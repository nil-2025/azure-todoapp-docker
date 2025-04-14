# A sample todo app in react
## DockerFile
```
FROM node:18-alpine AS installer
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
FROM nginx:latest AS deployer
COPY --from=installer /app/build /usr/share/nginx/html
```
# Commands
```
az acr login --name=$(containerRegistry)

az container create \
--name day10app \
--resource-group nbdemo \
--image $(containerRegistry)/$(imageRepository):$(tag) \
--registry-login-server $(containerRegistry) \
--registry-username nbdemocontainerregistry  \
--registry-password XXX \
--os-type Linux \
--dns-name-label aci-demo-nilotpal
```
# Azure DevOps CICD Pipeline to deploy to ACI
**Pipeline Code:**
```
trigger:
- main

resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: 'XXXX'
  imageRepository: 'todoapp'
  containerRegistry: 'NAME.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

  # Agent VM image name
  vmImageName: 'ubuntu-latest'

stages:
- stage: Build
  displayName: Build and push stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'XXXX'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: 'az acr login --name=$(containerRegistry)'
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'XXXX'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az container create \
          --name day10app \
          --resource-group day10-demo \
          --image $(containerRegistry)/$(imageRepository):$(tag) \
          --registry-login-server $(containerRegistry) \
          --registry-username day10demo  \
          --registry-password XXXX \
          --dns-name-label aci-demo-nilotpal
```
