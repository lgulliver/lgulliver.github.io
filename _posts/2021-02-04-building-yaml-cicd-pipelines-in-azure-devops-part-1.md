---
author: Liam Gulliver
title: Building YAML CI/CD Pipelines in Azure DevOps [Part 1]
tags: devops azure-devops continuous-delivery continuous-integration continuous-deployment ci/cd
date: 2021-02-04 18:00:00
header:
  video:
    id: JW9PLJTQ0No
    provider: youtube
---

Recently, [Azureish Live!](https://twitch.tv/azureishlive) returned - our new project is to build a zero touch CI/CD pipeline in [Azure DevOps](https://lgulliver.github.io/tags/#azure-devops) for the [project we build last year for the show](https://lgulliver.github.io/azureish-live!-building-a-git-commit-watcher-with-azure-functions-and-github-part1/).

You can watch the whole first part of the show above. Other than a minor technical hiccup, I think we did pretty well!

In this post, we're going to take a quick look at what we've got to do and what we achieved in our first part. 

What we created as part of our GitWatcher project was a complete serverless solution using Azure Functions, Azure SignalR and Azure Storage for hosting a static website build with Blazor WebAssembly.

The application currently looks like the below:

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="319px" viewBox="-0.5 -0.5 319 187" content="&lt;mxfile host=&quot;app.diagrams.net&quot; modified=&quot;2021-02-04T15:03:31.699Z&quot; agent=&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36 Edg/88.0.705.56&quot; etag=&quot;xrGKU2o0LIArBVjX5Qau&quot; version=&quot;14.2.9&quot;&gt;&lt;diagram id=&quot;rnFcCY4i2SYY_jUFPXYP&quot; name=&quot;Page-1&quot;&gt;3Vddb5swFP01PG4KH6btY77aTVrVaJnUx8kFFzwZLjKXBPrrdykmhKJEtGu3LE/Y517j63POxcJy50l5o3kW30IolOVMwtJyF5bj2I5/RY8aqQwymTgNEmkZGqwD1vJJtIkGLWQo8l4iAiiUWR8MIE1FgD2Maw3bftojqP6uGY/EAFgHXA3Rexli3KCO6/pd4IuQUdxu7TOviSS8zTZHyWMewnYPcpeWO9cA2IySci5UTV9LTLPu+kB0V5kWKY5ZAKqa3iArsuwuxmka/7izw09Gnw1XhTmxKRarlgIREiNmChpjiCDlatmhMw1FGop6mwnNupxvABmBNoG/BGJl5OUFAkExJspEmz3rjQ6ezUA5FDoQRw7UmoTrSOCRPG+nAJlXQCJQV7ROC8VRbvp1cGOiaJfX0UwDw/QrWLcHrE9XXwfE8zxrLP0oy5rdfcYykCk+l8VmFlsQwpWMUgICokxoAmTy7O3ZI6RomLedDl/IJKLilXyoj5AHXNDzukgDlJDmn/NNtBNmIzSK8rg0QyrNAtcz/jefAIexZr7t+omZlHivkzz/g8i/PDfLOyMtf0Cnv2N5Z8D6SkNZnanpPXZipreH/P/nrvdGup79oevN0lXtvT2BXdYT2L1i/Vc0dZlVL8TblfF2Pb2BnGvqBK6+19QJvZHE3Qm0linq424T92IyqrFa7N0b6+Lc+oqN7CvnX94mbMD6TPEn0ITdT9e3luMrqn32QIAf1aM1UlUBhdcST6MzEDRFf06DgMTHd7x72OXLu8cfd/ewV4tF0+7/pfm0df+B7vI3&lt;/diagram&gt;&lt;/mxfile&gt;" onclick="(function(svg){var src=window.event.target||window.event.srcElement;while (src!=null&amp;&amp;src.nodeName.toLowerCase()!='a'){src=src.parentNode;}if(src==null){if(svg.wnd!=null&amp;&amp;!svg.wnd.closed){svg.wnd.focus();}else{var r=function(evt){if(evt.data=='ready'&amp;&amp;evt.source==svg.wnd){svg.wnd.postMessage(decodeURIComponent(svg.getAttribute('content')),'*');window.removeEventListener('message',r);}};window.addEventListener('message',r);svg.wnd=window.open('https://viewer.diagrams.net/?client=1&amp;page=0&amp;edit=_blank');}}})(this);" style="cursor:pointer;max-width:100%;max-height:187px;"><defs/><g><path d="M 41 46 L 41 108.63" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 41 113.88 L 37.5 106.88 L 41 108.63 L 44.5 106.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><image x="15.5" y="-0.5" width="50" height="46" xlink:href="https://app.diagrams.net/img/lib/mscae/Functions.svg"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 53px; margin-left: 41px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: white; white-space: nowrap; ">API</div></div></div></foreignObject><text x="41" y="65" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">API</text></switch></g><path d="M 126 23 L 72.37 23" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 67.12 23 L 74.12 19.5 L 72.37 23 L 74.12 26.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><image x="125.5" y="-0.5" width="50" height="46" xlink:href="https://app.diagrams.net/img/lib/mscae/Functions.svg"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 53px; margin-left: 151px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: white; white-space: nowrap; ">Proxy</div></div></div></foreignObject><text x="151" y="65" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Proxy</text></switch></g><path d="M 66 140 L 281 140 L 281 52.37" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 281 47.12 L 284.5 54.12 L 281 52.37 L 277.5 54.12 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><image x="15.5" y="114.5" width="50" height="50" xlink:href="https://app.diagrams.net/img/lib/mscae/SignalR.svg"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 172px; margin-left: 41px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: white; white-space: nowrap; ">SignalR service</div></div></div></foreignObject><text x="41" y="184" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">SignalR...</text></switch></g><path d="M 256 23.5 L 216 23.5 L 182.37 23.08" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 177.12 23.01 L 184.16 19.6 L 182.37 23.08 L 184.07 26.6 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><image x="255.5" y="0.5" width="50" height="45" xlink:href="https://app.diagrams.net/img/lib/mscae/Storage_Accounts.svg"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe flex-start; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 53px; margin-left: 281px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: white; white-space: nowrap; ">Blazor WASM<br />Static Site</div></div></div></foreignObject><text x="281" y="65" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Blazor W...</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>

The Blazor static site is ultimately something that needs to be compiled, as is the API we have. We chose to start with the API as the easiest part to build. Most importantly, for now, we have chosen only to build it and not deploy just yet to keep things simple.

The API is a simple Function App with a couple of methods to be called by the front-end. You can see the code we wrote for that over at the [GitHub repo](https://github.com/AzureishLive/gitwatcher/tree/main/backend).

It is also important to note that for this project we have a single repository that contains all components. Our approach also means that we will be having one YAML pipeline per component, all in the same repository.

A couple of constraints we set ourselves:

1. We want a build to run for _every_ commit.
2. We won't be running tests because we haven't written any (yet).

Getting setup with a stage for `build` along with the relevant triggers for our GitHub Flow branching strategy is relatively straightforward. We need to define our triggers, agent pool and of course, the first stage.

```yaml
trigger:
- main
- feature/*

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: build

    jobs:
      - job: build
        steps:
```        

The triggers set here means that a build will run any time a commit is pushed (or merged into) `main` along with any branches that appear under `feature/*` e.g. `feature/adding-pipelines`.

Our first task is to ensure we have the right SDK for dotnet core available to us - in our case, this is 3.1.x LTS as we are yet to move GitWatcher to .NET 5 (but this will happen in a future show).

To do this, we need to use the `UseDotNet` task. This task allows us to explicitly define the SDK (or runtime) version of dotnet we wish to use. We now have to include this step as the default version available on the Microsoft-hosted agents is now .NET 5.

It doesn't have to be exact, it can be a latest minor version which is what we do here by using the following configuration:

```yaml
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '3.1.x'
  displayName: 'Set SDK to 3.1.x'
``` 

Now that we have that, the next thing we needed to do was to actually build the project!

As I mentioned, we haven't written tests for this, even though we probably should!

```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '$(Build.SourcesDirectory)/backend/backend.csproj'
  displayName: 'Build backend service'
```

What we're doing here is being specific about the project we want to build. As I mentioned earlier, what we have is a single repo and we don't want to build the entire solution all at once.

Our next step is to create a package to publish:

```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    projects: '$(Build.SourcesDirectory)/backend/backend.csproj'
    arguments: '-o $(Build.ArtifactStagingDirectory)'
  displayName: 'dotnet publish'
```

We've set the output of the publish action to put in the `ArtifactStagingDirectory`. This is where we'll publish our artifact from to our build for later use.

Ultimately, all this command is running is the `dotnet publish` command.

Then our final step is to publish the build artifact:

```yaml
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```

But what if we don't want to create a build artifact every time? In my opinion, we shouldn't need to. At least not when we're building from feature branches on every commit for validation. We also don't need to take up the storage for it every time we do a build and it's not form `main` (we can though, nothing stopping you doing that, it's just not my preferred practice).

On all the steps we don't want to run depending on the source branch, we added a `condition`. For this particular build definition, we added the `condition` to both the `dotnet publish` and `PublishBuildArtifact` tasks.

The condition itself, looks a little like this:

```yaml
condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

This is saying that providing the previous step succeeded, and the branch that triggered the build is `main` then we'll run this step.

The complete step with the condition looks like this:

```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    projects: '$(Build.SourcesDirectory)/backend/backend.csproj'
    arguments: '-o $(Build.ArtifactStagingDirectory)'
  displayName: 'dotnet publish'
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

Putting the whole thing together gives us a YAML pipeline that at this stage allows us to do continuous integration builds on every commit.

```yaml
trigger:
- main
- feature/*

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: build

    jobs:
      - job: build
        steps:
        - task: UseDotNet@2
          inputs:
            packageType: 'sdk'
            version: '3.1.x'
          displayName: 'Set SDK to 3.1.x'

        - task: DotNetCoreCLI@2
          inputs:
            command: 'build'
            projects: '$(Build.SourcesDirectory)/backend/backend.csproj'
          displayName: 'Build backend service'

        - task: DotNetCoreCLI@2
          inputs:
            command: 'publish'
            publishWebProjects: false
            projects: '$(Build.SourcesDirectory)/backend/backend.csproj'
            arguments: '-o $(Build.ArtifactStagingDirectory)'
          displayName: 'dotnet publish'
          condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))

        - task: PublishBuildArtifacts@1
          inputs:
            PathToPublish: '$(Build.ArtifactStagingDirectory)'
            ArtifactName: 'drop'
            publishLocation: 'Container'
          condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

You'll perhaps notice though that currently, there's one glaring omission that means that this build will run when _any_ commit is pushed back to the repo, regardless of project.

On the next show, we'll be demonstrating how to solve that and moving on to infrastructure as code!