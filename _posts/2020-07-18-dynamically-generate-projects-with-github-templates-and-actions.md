---
author: Liam Gulliver
title: Dynamically Create Projects With Github Templates and Actions
tags: github github-actions workflow
date: 2020-07-18 21:50:00
---

This week I've been experimenting with template repositories in Github. There's a lot of really great stuff you can do to scaffold your projects out of the box with them, but I wanted to take it one step further and have a template that only contained the bare minimum and no pre-existing project files other than a `.gitignore` and the relevant workflow.

To be clear, my goal here is to have the project generated when the repository is created. I'm going to use .NET Core for this particular example, but I think the solution should work for most other languages too, provided it has a CLI.

Here's the repository as it stands at time of writing for this post:

```
|   .gitignore
|   LICENSE
\---.github
    |   dependabot.yml  
    \---workflows
            ci.yml
            init.yml
```            

As you can see the [template repository](https://github.com/lgulliver/dotnet-api-template) only contains the license, `.gitignore` configured for the project type, my workflows and I've also configured Dependabot. This particular repository will create a brand new .NET Core 3.1.x Web API project, complete with a unit test project too.

I've also got a CI workflow which is simply the starter .NET Core action workflow for building .NET Core projects.

## The Code

I have a workflow called `init.yml`. This is the workflow that runs when a new repository is created and will create the project.

The great thing about Github Actions is that we can also set conditionals. What I've done here is set it so that the entire job will only run of the branch is on `master` and the trigger is `create`. As an aside, this currently runs when other branches are created, but the job labelled `build` will only run if its the `master` branch, so essentially nothing happens unless the branch being created is `master`. There might be some other edge cases that might trigger it and run all the actions but I've not come across them yet.

So let's break it down.

### Setting up the workflow

As I mentioned, I want this to run on a `create` event.

```yaml
name: Init Project

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on: create
```

### Run the job only if the branch is master

The `if:` condition is set on the entire job labelled `build`. I'm using the environment variable `github.ref` and checking that it is equal to the ref for `master`.

Anything that doesn't meet that criteria will mean that the whole job is ignored.

```yaml
# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    if: github.ref == 'refs/heads/master'
    
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
```

### Setup .NET Core projects

At this point, the workflow is running the standard steps to configure it for .NET Core

```yaml
# Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v1.5.0
      with:
        # SDK version to use. Examples: 2.2.104, 3.1, 3.1.x
        dotnet-version: 3.1.x
```      

Here's where we start doing things that for me at least, wouldn't be expected as part of a workflow - creating the projects itself. There's nothing fancy here really but one of the reasons I wanted to do this as part of the template repo as I actually prefer this way to have it setup all the right namespaces. It's plenty enough for creating a basic Web API project and then any special needs for a specific project I can go ahead and add to that project.

What I am doing here though is naming the projects after the repository name, excluding my Github user name by using `github.event.repository.name`. This provides the repo name as part of the event metadata for the workflow.

```yaml
    - name: Generate project
      run: dotnet new webapi --name {% raw %}${{ github.event.repository.name }}{% endraw %}
     
    - name: Generate test project
      run: dotnet new xunit --name {% raw %}${{ github.event.repository.name }}.UnitTests{% endraw %}
```      

### Add, Commit and Push Changes as part of the workflow

Of course what we've done here is create a bunch of files that we actually want to commit back to our repo. To do this, I use the `actions-x/commit` action. You can add custom messages and all sorts, but I'm quite happy with the defaults.

```yaml
    - name: Git Commit/Push Changes
      uses: actions-x/commit@v1
```

### Bringing it all together

```yaml
name: Init Project

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on: create

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    if: github.ref == 'refs/heads/master'
    
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v1.5.0
      with:
        # SDK version to use. Examples: 2.2.104, 3.1, 3.1.x
        dotnet-version: 3.1.x
    
    - name: Generate project
      run: dotnet new webapi --name {% raw %}${{ github.event.repository.name }}{% endraw %}
     
    - name: Generate test project
      run: dotnet new xunit --name {% raw %}${{ github.event.repository.name }}.UnitTests{% endraw %}
    
    - name: Git Commit/Push Changes
      uses: actions-x/commit@v1
```      

## Using the template repository

First of all, we need to tell Github the repository is a template repo. This is done by going to **Settings** and checking the **Template repository** box.

{% include figure image_path="/assets/images/posts/template-repo-setting-github.png" alt="Template repository setting" %}

Now that means we can use this to create new repositories from.

{% include figure image_path="/assets/images/posts/pick-template-repository-github.png" alt="Pick a template" %}

Once you've named your new repo and hit the **Create Repository** button, a new repo will be created containing just the files from the template repository at this point.

```
|   .gitignore
|   LICENSE
\---.github
    |   dependabot.yml  
    \---workflows
            ci.yml
            init.yml
```  

It'll take a minute or two but the Init Project workflow will run. You can check on it under the **Actions** tab.

{% include figure image_path="/assets/images/posts/template-repo-running-init.png" alt="Project initialization" %}

Once it's completed, a new commit will be generated in the repository with the new .NET Core project files created too.

{% include figure image_path="/assets/images/posts/init-complete-github.png" alt="Projects created" %}

The new file structure should look a little something like this:

```
|   .gitignore
|   LICENSE
|
+---.github
|   |   dependabot.yml
|   |
|   \---workflows
|           ci.yml
|           init.yml
|
+---ThisIsMy.API
|   |   appsettings.Development.json
|   |   appsettings.json
|   |   Program.cs
|   |   Startup.cs
|   |   ThisIsMy.API.csproj
|   |   WeatherForecast.cs
|   |
|   +---Controllers
|   |       WeatherForecastController.cs
|   |
|   \---Properties
|           launchSettings.json
|
\---ThisIsMy.API.UnitTests
        ThisIsMy.API.UnitTests.csproj
        UnitTest1.cs
```

There might be a better way to do this, but it seems to do the trick for me without having to work too hard to get it going.