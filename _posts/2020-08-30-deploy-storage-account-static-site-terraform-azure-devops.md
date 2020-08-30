---
author: Liam Gulliver
title: Deploying a Static Website to Azure Storage with Terraform and Azure DevOps
tags: devops terraform static-site azure-storage azure azure-devops
date: 2020-08-30 12:30:00
tagline: "A quick example on using Terraform to deploy a static website enabled storage account to Azure and deploy the content from Azure DevOps"
header:
  image: /assets/images/featured/tflogo.png 
---

This week I've been working on using static site hosting more as I continue [working with Blazor](https://lgulliver.github.io/serverless-chat-with-blazor-webassembly/) on some personal projects.

My goal is to deploy a static site to Azure, specifically into an Azure Storage account to host my site, complete with Terraform for my infrastructure as code. Now, I'll caveat this with saying Terraform isn't my strong point but this seems to work for me and I hope you find it useful too. I'm also going to use Terraform 0.12 for this post rather than 0.13 as I've not gone through changes and so on yet for it.

As always, if you'd like to skip straight to the code, you can find it in the [lgulliver/TerraformAzureStorageStaticSite repository on GitHub](https://github.com/lgulliver/TerraformAzureStorageStaticSite).

## Outline of workflow

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="791px" viewBox="-0.5 -0.5 791 91" content="&lt;mxfile host=&quot;app.diagrams.net&quot; modified=&quot;2020-08-30T11:17:58.252Z&quot; agent=&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36 Edg/85.0.564.41&quot; etag=&quot;_8prRKhlgFddguKSxBQh&quot; version=&quot;13.6.5&quot;&gt;&lt;diagram id=&quot;nlnsHCpNwOT_Ug9kOCka&quot; name=&quot;Page-1&quot;&gt;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&lt;/diagram&gt;&lt;/mxfile&gt;" onclick="(function(svg){var src=window.event.target||window.event.srcElement;while (src!=null&amp;&amp;src.nodeName.toLowerCase()!='a'){src=src.parentNode;}if(src==null){if(svg.wnd!=null&amp;&amp;!svg.wnd.closed){svg.wnd.focus();}else{var r=function(evt){if(evt.data=='ready'&amp;&amp;evt.source==svg.wnd){svg.wnd.postMessage(decodeURIComponent(svg.getAttribute('content')),'*');window.removeEventListener('message',r);}};window.addEventListener('message',r);svg.wnd=window.open('https://viewer.diagrams.net/?client=1');}}})(this);" style="cursor:pointer;max-width:100%;max-height:91px;"><defs/><g><rect x="650" y="0" width="140" height="90" rx="13.5" ry="13.5" fill="#ffffff" stroke="#000000" pointer-events="all"/><path d="M 570 45 L 643.63 45" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 648.88 45 L 641.88 48.5 L 643.63 45 L 641.88 41.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="430" y="0" width="140" height="90" rx="13.5" ry="13.5" fill="#ffffff" stroke="#000000" pointer-events="all"/><path d="M 140 45 L 203.63 45" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 208.88 45 L 201.88 48.5 L 203.63 45 L 201.88 41.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="0" y="0" width="140" height="90" rx="13.5" ry="13.5" fill="#ffffff" stroke="#000000" pointer-events="all"/><path d="M 75.06 52.5 C 61.78 52.5 52.5 41.68 52.5 30.17 C 52.5 16.41 63.58 7.5 75.22 7.5 C 86.43 7.5 97.5 16.7 97.5 29.99 C 97.5 41.9 88.03 52.5 75.06 52.5 Z" fill="#0d2636" stroke="none" pointer-events="all"/><path d="M 70.19 47.57 C 70.19 48.55 69.45 48.95 68.35 48.57 C 61.69 46.24 55.28 39.03 55.28 30.03 C 55.28 17.86 65.7 10.18 74.72 10.18 C 86.44 10.18 94.77 19.78 94.77 29.91 C 94.77 38.27 89.39 45.97 81.33 48.67 C 80.41 48.91 79.85 48.43 79.85 47.63 L 79.85 41.77 C 79.85 40.63 79.38 39.43 78.53 38.59 C 81.81 38.21 83.79 37.39 85.32 35.83 C 86.81 34.34 87.48 32.17 87.6 29.48 C 87.69 27.34 87.12 25.26 85.57 23.67 C 86.11 22.36 86.2 20.62 85.38 18.47 C 83.76 18.35 81.92 19.32 80.03 20.49 C 76.7 19.62 73.38 19.51 70.06 20.54 C 68.57 19.57 67.23 18.49 64.65 18.47 C 63.97 20.37 63.86 22.1 64.45 23.64 C 62.61 25.68 62.4 27.68 62.43 29.68 C 62.63 33.54 64.04 35.49 65.61 36.65 C 66.91 37.61 68.74 38.22 71.52 38.63 C 70.77 39.37 70.35 40.25 70.28 41.28 C 68.65 42.02 66.31 42.4 64.71 40.13 C 63.99 38.99 63.02 37.7 61.17 37.71 C 60.86 37.7 60.56 37.82 60.52 37.95 C 60.48 38.09 60.65 38.39 60.85 38.51 C 62.47 39.54 62.74 40.12 63.4 41.53 C 64.01 43.18 65.08 43.81 66.23 44.26 C 67.41 44.67 69.31 44.56 70.19 44.26 Z" fill="#ffffff" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 60px; margin-left: 75px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; ">Push to GitHub</div></div></div></foreignObject><text x="75" y="72" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Push to...</text></switch></g><image x="694.5" y="7" width="50" height="45" xlink:href="https://app.diagrams.net/img/lib/mscae/Storage_Accounts.svg"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 60px; margin-left: 720px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: white; white-space: nowrap; ">Deploy to Blob Storage</div></div></div></foreignObject><text x="720" y="72" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Deploy t...</text></switch></g><image x="474.75" y="2.25" width="49.5" height="49.5" xlink:href="data:image/png;base64,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" preserveAspectRatio="none"/><g fill="#000000" font-family="Helvetica" text-anchor="middle" font-size="12px"><rect fill="#ffffff" stroke="none" x="444" y="60" width="113" height="29" stroke-width="0"/><text x="499.5" y="69.75">Deploy infrastructure</text><text x="499.5" y="83.75">with Terraform</text></g><path d="M 350 45 L 423.63 45" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 428.88 45 L 421.88 48.5 L 423.63 45 L 421.88 41.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="210" y="0" width="140" height="90" rx="13.5" ry="13.5" fill="#ffffff" stroke="#000000" pointer-events="all"/><image x="264.5" y="7" width="40" height="40" xlink:href="https://app.diagrams.net/img/lib/mscae/Azure_Pipelines&#9;.svg"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 55px; margin-left: 285px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: white; white-space: nowrap; ">Trigger Azure <br />DevOps pipeline</div></div></div></foreignObject><text x="285" y="67" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Trigge...</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://desk.draw.io/support/solutions/articles/16000042487" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>

This is a highly simplified version of what I want to do. I'm going to keep my code in GitHub (being completely honest here, it's partly because I can then play with the new Codespaces beta), which will trigger an Azure DevOps pipeline to do three things:

1. Deploy any required infrastructure to host my static site (Azure Storage Account)
2. Deploy my static files to the infrastructure (HTML page)
3. Define two environments, dev and prod.

I'm also going to stick to an "everything as code" approach here so that'll include my pipeline as well.

As for this example I'm just moving a HTML page around, we won't need to do a build here.

## Setting up the Terraform backend

My pipeline is made up of two stage, dev and prod, but before we get started, we also need to create the infrastructure for the terraform backend. 

I'm almost 100% certain there's a better way than this, but what I've done here is created an ARM template to create the storage account that will store the Terraform state. It's created with a partially randomly generated name to ensure uniqueness.

The ARM template also creates the blob storage container in the storage account.

It will be the first thing we run in the pipeline. 

I've created this file under `infrastructure/backend/tfbackend.deploy.json`. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat('tfstate', uniquestring(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "properties": {},
      "resources": [
        {
          "type": "blobServices/containers",
          "apiVersion": "2019-06-01",
          "name": "default/tfstate",
          "dependsOn": [
            "[variables('storageAccountName')]"
          ]
        }
      ]
    }
  ],
  "outputs": {
    "storageAccountName": {
      "type": "string",
      "value": "[variables('storageAccountName')]"
    }
  }
}
```

If there's a better way to do this, please let me know! I'm 100% guilty of sticking to what I know here for berevity.

## Defining the Terraform

For the Terraform part of the solution, I've created 4 files to handle the core infrastructure, anything that may vary from environment to environment and then I've also stuck provider versions.

### `az-storage-account-main.tf`

This file contains the definition for the infrastructure I want to create.

```terraform
terraform {
  backend "azurerm" {}
}

locals {
    env_prefix = "${var.shortcode}-${var.product}-${var.envname}-${var.location_short_code}"
    env_prefix_no_separator = "${var.shortcode}${var.product}${var.envname}${var.location_short_code}"
}

resource "azurerm_resource_group" "rg" {
  name     = "${local.env_prefix}-rg"
  location = var.location

  tags = {
    product = var.product      
  }
}

resource "azurerm_storage_account" "static_storage" {
  name                     = "${local.env_prefix_no_separator}stor"
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_kind             = "StorageV2"
  account_tier             = "Standard"
  account_replication_type = "GRS"
  enable_https_traffic_only = true

  static_website {
    index_document = "index.html"
  }

  tags = {
    product = var.product
  }
}
```

The important parts here are that I need to be using a `StorageV2` storage account.

All that is needed to enable the static site hosting on the storage account is to include the `static_website` block in the storage account definition. I've also gone ahead and defined what my index document for my static site is.

```terraform
  static_website {
    index_document = "index.html"
  }
```

When this isn't included, static site hosting is disabled as per the default behaviour.

### `az-storage-account-variables.tf`

This contains the variables I want to use for values that will change between environments and it defines their types.

```terraform
variable "location" { #Location of the Azure resources
    type = string
}
variable "location_short_code" { #3-character short code for location for naming
    type = string
}
variable "shortcode" { #buisiness unit short code
    type = string
}
variable "product" { #name of product or service being deployed
    type = string
}
variable "envname" { #name of the environment being deployed
    type = string
}
```

### `az-storage-account-variables.tfvars`

This is the equivalent of a parameters file in ARM. I'm going to be doing some token replacement here for each stage.

```terraform
location = "__location__"
location_short_code = "__location_short_code__"
shortcode = "__shortcode__"
product = "__product__"
envname = "__environment_name__"
```

### `versions.tf`

Finally, I have a version file that sticks providers and modules to specific versions. In particular for this solution I need version 2.2.0 or greater of the `azurerm` provider, but I'm going to stick it to 2.2.0.

```terraform
terraform {
    required_version = "~> 0.12.29"
}

provider "azurerm" {
    version = "~>2.2.0"
    features {}
}
```

## Defining the pipeline

Now that I've got my code for both my Terraform backend and my storage account to host my site, I need to define my build and release pipeline for Azure DevOps.

As I've only got a [single HTML file](https://github.com/lgulliver/TerraformAzureStorageStaticSite/blob/master/code/index.html) for this example, I'm going to skip defining a build here.

I mentioned earlier on in this post that I was aiming for two environments, dev and prod. 

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="321px" viewBox="-0.5 -0.5 321 61" content="&lt;mxfile host=&quot;app.diagrams.net&quot; modified=&quot;2020-08-30T11:52:00.164Z&quot; agent=&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36 Edg/85.0.564.41&quot; etag=&quot;D19TXfu4fPlj-D_GxuPA&quot; version=&quot;13.6.5&quot;&gt;&lt;diagram id=&quot;nlnsHCpNwOT_Ug9kOCka&quot; name=&quot;Page-1&quot;&gt;xZRNU4MwEEB/DcfOQLDYXq2tHuypM/ackpXgBJZJl6/+ekMJTRm0owf1BHnZsNmXDV64yponzQu5RQHKY75ovPDRYyxgzDePjrSW+MuoJ4lOhWUO7NITDIGWlqmA4yiQEBWlxRjGmOcQ04hxrbEeh72hGmcteAITsIu5mtJ9Kkj2dMHuHX+GNJFD5iBa9jMZH4JtJUfJBdZXKFx74UojUv+WNStQnb3BS79u88XsZWMacvrOgm2rNnoxf93AabbKovqwbw4zFvafqbgqbcV2t9QOCkAYI3aImiQmmHO1dvRBY5kL6PL4ZuRiXhALAwMD34GotcfLS0KDJGXKzvY5u0RfFmfREUsdw62Khi7hOgG6Fcguh2DaFzAD0q1ZqEFxSqvxTrhto+QS50ybFyv7J+KDiXgB1cS9M9tpqmVKsCv4WUBtbtxnFivQBM1tj9Oq7YK5bVZ7Xe+Wdly73g+GhpZXfR/5v+WJTTwVGsW/i/pLU2bofhPnuau/bbj+AA==&lt;/diagram&gt;&lt;/mxfile&gt;" onclick="(function(svg){var src=window.event.target||window.event.srcElement;while (src!=null&amp;&amp;src.nodeName.toLowerCase()!='a'){src=src.parentNode;}if(src==null){if(svg.wnd!=null&amp;&amp;!svg.wnd.closed){svg.wnd.focus();}else{var r=function(evt){if(evt.data=='ready'&amp;&amp;evt.source==svg.wnd){svg.wnd.postMessage(decodeURIComponent(svg.getAttribute('content')),'*');window.removeEventListener('message',r);}};window.addEventListener('message',r);svg.wnd=window.open('https://viewer.diagrams.net/?client=1');}}})(this);" style="cursor:pointer;max-width:100%;max-height:61px;"><defs/><g><path d="M 120 30 L 193.63 30" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 198.88 30 L 191.88 33.5 L 193.63 30 L 191.88 26.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="0" y="0" width="120" height="60" rx="9" ry="9" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 1px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; ">dev</div></div></div></foreignObject><text x="60" y="34" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">dev</text></switch></g><rect x="200" y="0" width="120" height="60" rx="9" ry="9" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 118px; height: 1px; padding-top: 30px; margin-left: 201px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; ">prod</div></div></div></foreignObject><text x="260" y="34" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">prod</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://desk.draw.io/support/solutions/articles/16000042487" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>

To achieve this, I'm going to use the stages capability in Azure DevOps YAML.

A basic setup of stages could look something a little like this:

```yaml
stages:
- stage: dev
  jobs:
  - job: infrastructure
    pool: 'ubuntu-latest'

    steps:
    # tasks go here

- stage: prod
  dependsOn: dev
  jobs:
  - job: infrastructure
    pool: 'ubuntu-latest'

    steps:
    # tasks go here
```

Stages can each have their own variables too as well as use globally defined variables in the pipeline. They can also have approval gates enabled, but for the purpose of this example, I'm not going to do that and I'm going to go ahead and deploy automatically.

In each stage I'm going to execute the following:

1. Deployment of ARM for Terraform backend.
2. Replace tokens in my `tfvars`.
3. Ensure I have the version of Terraform I want to use (0.12.29) available on the build agent
4. `terraform init`
5. `terraform plan`
6. `terraform apply`
7. Deploy to Azure storage

The first thing I'm going to do is create myself a service connection in Azure DevOps to my Azure subscription so that I can deploy. Take a look at the [official docs](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml) on how to go about that.

As I don't want to store the name of that service connection or any subscription IDs in my repository, I'm going to store those as variables in Azure DevOps.

{% include figure image_path="/assets/images/posts/azuredevops_vars.png" alt="Azure DevOps variables" %}

Now I've done that, I can start putting together my pipeline YAML starting with my global variables that don't need to be secret or are values I want to keep wherever I go.

```yaml
trigger:
- master

variables:
  resource_group_tfstate: 'tfstate-uks-rg'
  product: 'staticsite'
  shortcode: 'lg'  

stages:
```

You'll notice I don't define an agent pool here either, that's because I do it at the job level as one of the tasks we use later is currently only supported on Windows agents.

So let's define our first stage, dev:

```yaml
- stage: dev
  variables:    
    location: 'uksouth'        
    environment_name: 'dev'
    location_short_code: 'uks'
    backendAzureRmContainerName: tfstate
    backendAzureRmKey: tfdev 
  jobs: 
```

Dev doesn't depend on any pre-requisite steps that it depends on before the stage can be executed. Under normal cricumstance, I would potentially have a build as the first step which my dev stage would then depend on.

What I do have here though are my stage specific variables. Some are the same in both stages but I've defined them in each as they are the most likely to change.

The next thing I'm going to do is define the first job that will be executed for the stage which is to setup the infrastructure.

```yaml
jobs:
  - job: Infrastructure
    displayName: 'Infrastructure'
    pool:
      vmImage: 'ubuntu-latest'
```

This is naming the job and setting what agent pool I wish to use.

As I mentioned earlier, my first step is to deploy the ARM template for the backend. I'm going to run this same step for each environment too using the Azure Resource Manager Template Deployment task.

```yaml
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
```

This will create a resource group solely for the shared backend storage account if it doesn't already exist, then create or update the storage account along with the blob containers required as defined by the ARM template.

You may have noticed in the ARM template I define an output. This can be quite tricky to retrieve on an agent as the default task outputs it to JSON which you then have to parse. Thankfully, there's an ARM outputs task which will do all the heavy lifting for you and provide you with a pipeline variable that is named the same as your output.

```json
"outputs": {
    "storageAccountName": {
      "type": "string",
      "value": "[variables('storageAccountName')]"
    }
  }
```

As you can see above, it creates an output called `storageAccountName` which the ARM outputs task will then use to create a pipeline variable with the same name that I can then use as `$(storageAccountName)`.

This variable contains the name of the storage account for the Terraform backend where I store my state.

```yaml
    - task: ARM Outputs@6
      inputs:
        ConnectedServiceNameSelector: 'ConnectedServiceNameARM'
        ConnectedServiceNameARM: '$(armConnection)'
        resourceGroupName: '$(resource_group_tfstate)'      
        whenLastDeploymentIsFailed: 'fail'
```

From here, I'm going to do my other pre-requisite task which is to replace tokens in my `tfvars` file that I have defined with double underscores, with the appropriate values from the pipeline. To do this, I'm going to use the Replace Tokens task.

```yaml
    - task: qetza.replacetokens.replacetokens-task.replacetokens@3
      displayName: 'Replace tokens in **/*.tfvars'
      inputs:
        rootDirectory: '$(System.DefaultWorkingDirectory)/infrastructure/storage-account'
        targetFiles: '**/*.tfvars'
        escapeType: none
        tokenPrefix: '__'
        tokenSuffix: '__'
        enableTelemetry: false
```

The last setup task for the stage is to ensure we have the right Terraform version available. To do that, I use the Terraform Installer task.

```yaml
    - task: TerraformInstaller@0
      displayName: 'Install Terraform 0.12.29'
      inputs:
        terraformVersion: 0.12.29
```        

At this point, the pipeline YAML looks a little something like this:

```yaml
trigger:
- master

variables:
  resource_group_tfstate: 'tfstate-uks-rg'
  product: 'staticsite'
  shortcode: 'lg'  

stages:
- stage: dev
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
```      

I'm defining my global variables, the dev stage and the pre-requistes are deployed or configured ahead of executing the Terraform.

Executing the Terraform is broken down into 3 steps, `init`, `plan` and `apply`.

```yaml
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
```        

These steps will create an environment specific resource group and deploy the required resources into it.

Now, I need to create another job. The Azure File Copy job is by far the easiest way to deploy files into a blob container. It uses `azcopy` to copy the files, however, the task itself will only execute on a Windows agent which is why we need to create a second job.

```yaml
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

The deploy job also needs the required infrastructure to exist first, so I've set the `dependsOn` for the entire job to be the `Infrastructure` job which runs the Terraform.

Static website enabled storage accounts also require the files to be deployed into a specific blob container called `$web`. It gets created automatically when the static website option is enabled on the storage account.

Bringing it all together, the pipeline now looks like this:

```yaml
trigger:
- master

variables:
  resource_group_tfstate: 'tfstate-uks-rg'
  product: 'staticsite'
  shortcode: 'lg'  

stages:
- stage: dev
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
```

Should I execute that right now, it'll run and deploy my `index.html` file to the storage account.

{% include figure image_path="/assets/images/posts/staticsite-dev.png" alt="Hello there" %}

## Adding a second stage

To add in the prod stage, I'm going to run all of the same tasks again but with couple of environment specific variable values and one important addition to the stage definition.

```yaml
- stage: prod
  dependsOn: dev

  variables:    
    location: 'uksouth'        
    environment_name: 'prod'
    location_short_code: 'uks'
    backendAzureRmContainerName: tfstate
    backendAzureRmKey: tfprod
```

I'm making this stage dependent on dev succeeding and setting values where to `prod` where it was `dev` before.

From there, all tasks are identical to the previous step, which I can just copy and paste from the previous stage.

The overall pipeline YAML now looks like the below:

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

variables:
  resource_group_tfstate: 'tfstate-uks-rg'
  product: 'staticsite'
  shortcode: 'lg'  

stages:
- stage: dev
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

Heading into Azure DevOps once the pipeline has executed provides me with a view not too dissimilar to the Releases UI.

{% include figure image_path="/assets/images/posts/staticsite-pipeline.png" alt="YAML Pipeline" %}

I can also go into a job in a stage and see the output of each task.

{% include figure image_path="/assets/images/posts/staticsite-logexample.png" alt="YAML Pipeline Detail" %}

While only a basic setup here, I hope this helps to show you how to get up and running with static sites in Azure with Terraform and Azure DevOps.

## Resources

- [Azure DevOps Team Project](https://dev.azure.com/lgulliver/StaticSiteWithTerraform/)
- [GitHub repository for all code in this post](https://github.com/lgulliver/TerraformAzureStorageStaticSite/)