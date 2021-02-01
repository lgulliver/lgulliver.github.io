---
author: Liam Gulliver
title: Using Terrascan with Azure DevOps
tags: devops devsecops terrascan accurics azure-devops
date: 2021-01-21 18:00:00
---

[In my last post, I took a look at a new scanning tool called Terrascan.](https://lgulliver.github.io/scan-terraform-kubernetes-and-more-with-terrascan/) It can be used to ensure your Kubernetes manifests, Terraform and more are compliant with a set of built-in, or customised rules.

So far, my initial impressions of Terrascan have been positive (albeit, the release notes could use a little work). 

As you know, I'm a huge fan of [Azure DevOps](https://lgulliver.github.io/tags/#azure-devops) and one of the things I wanted to do with Terrascan is get it working as part of a CI/CD pipeline with the results output to Azure DevOps.

So let's take a look at that! 

Since my last delve into Terrascan, it has in fact been updated to 1.3.1 too, so I'll go ahead and use that. As an aside, it looks like they've now added another output format called "Human" too which is now the default. On the downside is that it still looks like the XML output isn't in a format that Azure DevOps agrees with so for now, I'm going to be content with the fact the task will still fail when issues are found.

For this post, I'm going to test using my Terraform from my post on [setting up an Azure Static Web Site with Terraform](https://lgulliver.github.io/deploy-storage-account-static-site-terraform-azure-devops/).

Taking the pipeline YAML from my prior post, I'm going to add in a validation stage at the start of the pipeline. This is where I'm going to do all my compliance checks before I do anything else.

These are the changes I've made at this point to add in a stage for validation:

```yaml
stages:
- stage: validate
  jobs:
  - job: Compliance
  displayName: 'Run Terrascan to check for compliance'
  pool: 
    vmImage: 'ubuntu-latest'


- stage: dev
  dependsOn: validate
```

Unfortunately, Terrascan doesn't currently have any marketplace extensions to add it to your CI/CD pipeline in Azure DevOps, but the great thing about Azure DevOps is you can practically install any tool you can think of to an agent. Even a Microsoft hosted one!

The way I'm going to do this here is by using the `script` task to pull down a specific version and install it.

```yaml
- script: |
    curl --location https://github.com/accurics/terrascan/releases/download/v1.3.1/terrascan_1.3.1_Linux_x86_64.tar.gz --output terrascan.tar.gz
    tar -xvf terrascan.tar.gz
    sudo install terrascan /usr/local/bin    
  displayName: 'Get tools'
```

In the above snippet, I'm getting version 1.3.1 of Terrascan from GitHub, extracting it and installing it to the agent. As I'm using the Microsoft hosted agents, this step will need to be run every build.

I also want to run Terrascan as soon as it has installed. Doing that will require another `script` task and I'm going to make sure that it will be run from the directory my Terraform sits in.

```yaml
- script: |
    terrascan scan -t azure -i terraform
  workingDirectory: $(System.DefaultWorkingDirectory)/infrastructure/storage-account
  displayName: 'Run terrascan' 
```  

In this snippet, I'm specifying that I'm going to use the Azure provider and the Terraform ruleset.

Bringing it all together looks a little something like this:

```yaml
trigger:
- master

variables:
  resource_group_tfstate: 'tfstate-uks-rg'
  product: 'staticsite'
  shortcode: 'lg'  

stages:
- stage: validate

  jobs:
  - job: Compliance
    displayName: 'Run Terrascan to check for compliance'
    pool: 
      vmImage: 'ubuntu-latest'
  
    steps:
    - script: |
        curl --location https://github.com/accurics/terrascan/releases/download/v1.3.1/terrascan_1.3.1_Linux_x86_64.tar.gz --output terrascan.tar.gz
        tar -xvf terrascan.tar.gz
        sudo install terrascan /usr/local/bin    
      displayName: 'Get tools'

    - script: |
        terrascan scan -t azure -i terraform
      workingDirectory: $(System.DefaultWorkingDirectory)/infrastructure/storage-account
      displayName: 'Run terrascan'      


- stage: dev
  dependsOn: validate
  variables:    
    location: 'uksouth'        
    environment_name: 'dev'
    location_short_code: 'uks'
    backendAzureRmContainerName: tfstate
    backendAzureRmKey: tfdev  

  jobs:
  - job: Infrastructure
    displayName: 'Infrastructure'
    pool:
      vmImage: 'ubuntu-latest'

    steps:
    - task: AzureResourceManagerTemplateDeployment@3
      displayName: 'ARM Template deployment: Resource Group scope'
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: '$(armConnection)'
        subscriptionId: '$(subscription_id)'
        action: 'Create Or Update Resource Group'
        resourceGroupName: '$(resource_group_tfstate)'
        location: '$(location)'
        templateLocation: 'Linked artifact'
        csmFile: '$(System.DefaultWorkingDirectory)/infrastructure/backend/tfbackend.deploy.json'
        deploymentMode: 'Incremental'

    - task: ARM Outputs@6
      inputs:
        ConnectedServiceNameSelector: 'ConnectedServiceNameARM'
        ConnectedServiceNameARM: '$(armConnection)'
        resourceGroupName: '$(resource_group_tfstate)'      
        whenLastDeploymentIsFailed: 'fail'

    - task: qetza.replacetokens.replacetokens-task.replacetokens@3
      displayName: 'Replace tokens in **/*.tfvars'
      inputs:
        rootDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        targetFiles: '**/*.tfvars'
        escapeType: none
        tokenPrefix: '__'
        tokenSuffix: '__'
        enableTelemetry: false

    - task: TerraformInstaller@0
      displayName: 'Install Terraform 0.12.29'
      inputs:
        terraformVersion: 0.12.29

    - task: TerraformTaskV1@0
      displayName: 'Terraform init'
      inputs:
        provider: 'azurerm'
        command: 'init'
        workingDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        commandOptions: '-backend-config=$(System.DefaultWorkingDirectory)/infrastructure/storage-account/az-storage-account-variables.tfvars'
        backendServiceArm: '$(armConnection)'
        backendAzureRmResourceGroupName: '$(resource_group_tfstate)'
        backendAzureRmStorageAccountName: '$(storageAccountName)'
        backendAzureRmContainerName: '$(backendAzureRmContainerName)'
        backendAzureRmKey: '$(backendAzureRmKey)'

    - task: TerraformTaskV1@0
      displayName: 'Terraform plan'
      inputs:
        provider: 'azurerm'
        command: 'plan'
        workingDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        commandOptions: '-var-file="$(System.DefaultWorkingDirectory)/infrastructure/storage-account/az-storage-account-variables.tfvars" --out=planfile'
        environmentServiceNameAzureRM: '$(armConnection)'
        backendServiceArm: '$(armConnection)'
        backendAzureRmResourceGroupName: '$(resource_group_tfstate)'
        backendAzureRmStorageAccountName: '$(storageAccountName)'
        backendAzureRmContainerName: $(backendAzureRmContainerName)
        backendAzureRmKey: '$(backendAzureRmKey)'

    - task: TerraformTaskV1@0
      displayName: 'Terraform apply'
      inputs:
        provider: 'azurerm'
        command: 'apply'
        workingDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        commandOptions: '-auto-approve planfile'
        environmentServiceNameAzureRM: '$(armConnection)'
        backendServiceArm: '$(armConnection)'
        backendAzureRmResourceGroupName: '$(resource_group_tfstate)'
        backendAzureRmStorageAccountName: '$(storageAccountName)'
        backendAzureRmContainerName: $(backendAzureRmContainerName)
        backendAzureRmKey: '$(backendAzureRmKey)'

  - job: Deploy
    displayName: 'Deploy'
    pool:
      vmImage: 'windows-latest'
    dependsOn: 'Infrastructure'

    steps:    
    - task: AzureFileCopy@3
      inputs:
        SourcePath: '$(System.DefaultWorkingDirectory)/code'
        azureSubscription: '$(armConnection)'
        Destination: 'AzureBlob'
        storage: '$(shortcode)$(product)$(environment_name)$(location_short_code)stor'
        ContainerName: '$web'

- stage: prod
  dependsOn: dev

  variables:    
    location: 'uksouth'        
    environment_name: 'prod'
    location_short_code: 'uks'
    backendAzureRmContainerName: tfstate
    backendAzureRmKey: tfprod

  jobs:
  - job: Infrastructure
    displayName: 'Infrastructure'
    pool:
      vmImage: 'ubuntu-latest'

    steps:
    - task: AzureResourceManagerTemplateDeployment@3
      displayName: 'ARM Template deployment: Resource Group scope'
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: '$(armConnection)'
        subscriptionId: '$(subscription_id)'
        action: 'Create Or Update Resource Group'
        resourceGroupName: '$(resource_group_tfstate)'
        location: '$(location)'
        templateLocation: 'Linked artifact'
        csmFile: '$(System.DefaultWorkingDirectory)/infrastructure/backend/tfbackend.deploy.json'
        deploymentMode: 'Incremental'

    - task: ARM Outputs@6
      inputs:
        ConnectedServiceNameSelector: 'ConnectedServiceNameARM'
        ConnectedServiceNameARM: '$(armConnection)'
        resourceGroupName: '$(resource_group_tfstate)'      
        whenLastDeploymentIsFailed: 'fail'

    - task: qetza.replacetokens.replacetokens-task.replacetokens@3
      displayName: 'Replace tokens in **/*.tfvars'
      inputs:
        rootDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        targetFiles: '**/*.tfvars'
        escapeType: none
        tokenPrefix: '__'
        tokenSuffix: '__'
        enableTelemetry: false

    - task: TerraformInstaller@0
      displayName: 'Install Terraform 0.12.29'
      inputs:
        terraformVersion: 0.12.29

    - task: TerraformTaskV1@0
      displayName: 'Terraform init'
      inputs:
        provider: 'azurerm'
        command: 'init'
        workingDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        commandOptions: '-backend-config=$(System.DefaultWorkingDirectory)/infrastructure/storage-account/az-storage-account-variables.tfvars'
        backendServiceArm: '$(armConnection)'
        backendAzureRmResourceGroupName: '$(resource_group_tfstate)'
        backendAzureRmStorageAccountName: '$(storageAccountName)'
        backendAzureRmContainerName: '$(backendAzureRmContainerName)'
        backendAzureRmKey: '$(backendAzureRmKey)'

    - task: TerraformTaskV1@0
      displayName: 'Terraform plan'
      inputs:
        provider: 'azurerm'
        command: 'plan'
        workingDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        commandOptions: '-var-file="$(System.DefaultWorkingDirectory)/infrastructure/storage-account/az-storage-account-variables.tfvars" --out=planfile'
        environmentServiceNameAzureRM: '$(armConnection)'
        backendServiceArm: '$(armConnection)'
        backendAzureRmResourceGroupName: '$(resource_group_tfstate)'
        backendAzureRmStorageAccountName: '$(storageAccountName)'
        backendAzureRmContainerName: $(backendAzureRmContainerName)
        backendAzureRmKey: '$(backendAzureRmKey)'

    - task: TerraformTaskV1@0
      displayName: 'Terraform apply'
      inputs:
        provider: 'azurerm'
        command: 'apply'
        workingDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        commandOptions: '-auto-approve planfile'
        environmentServiceNameAzureRM: '$(armConnection)'
        backendServiceArm: '$(armConnection)'
        backendAzureRmResourceGroupName: '$(resource_group_tfstate)'
        backendAzureRmStorageAccountName: '$(storageAccountName)'
        backendAzureRmContainerName: $(backendAzureRmContainerName)
        backendAzureRmKey: '$(backendAzureRmKey)'

  - job: Deploy
    displayName: 'Deploy'
    pool:
      vmImage: 'windows-latest'
    dependsOn: 'Infrastructure'

    steps:    
    - task: AzureFileCopy@3
      inputs:
        SourcePath: '$(System.DefaultWorkingDirectory)/code'
        azureSubscription: '$(armConnection)'
        Destination: 'AzureBlob'
        storage: '$(shortcode)$(product)$(environment_name)$(location_short_code)stor'
        ContainerName: '$web'        
```

Now I have my validation step before any deployment begins. If it passes, it will deploy to my dev environment. If it fails, no deployment will happen.

I can see in the pipeline that currently, I have a failure with my scan.

{% include figure image_path="/assets/images/posts/terrascan-pipeline.png" alt="Pipeline view" %}

Digging into the failed task, as expected, I have a policy violation:

```
Violation Details -
    
	Description    :	Ensure that Azure Resource Group has resource lock enabled
	File           :	az-storage-account-main.tf
	Line           :	10
	Severity       :	LOW
```

I could override this policy, but for the purpose of this post, I'm going to actually go ahead and fix the violation.

To fix this particular violation, all I need to do is add a resource lock to the resource group which I'm going to add to my `az-storage-account-main.tf`.

```hcl
resource "azurerm_management_lock" "rg" {
  name       = "rg-lock"
  scope      = azurerm_resource_group.rg.id
  lock_level = "CanNotDelete"
  notes      = "Locked for compliance"
}
```

A quick commit later and we're away!

{% include figure image_path="/assets/images/posts/terrascan-pipeline-success.png" alt="Pipeline view" %}