---
author: Liam Gulliver
title: Container Security Scanning with Trivy and GitHub Actions
tags: docker security scanning github-actions github devops trivy containers
date: 2020-07-08 13:43:00
---

[Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/) is great and one of my all time favourite tools for ALM, but in recent years and more so since Microsoft's acquisition of Github, the tooling available out of the box is getting better all of the time. One such feature is Github Actions, which is a lot like the pipelines as YAML feature available in Azure DevOps - with good reason as they share a lot of the same underlying wiring.

Further to my earlier post on using [Trivy with Azure DevOps](https://lgulliver.github.io/container-security-scanning-with-trivy-in-azure-devops/) I wanted to explore the differences with Github Actions in particular for an upcoming project I'm intending to start work on.

## Using Trivy with Github Actions

There's a couple of different options for running Trivy with Github Actions, but for this we're going to focus on Aqua's own experimental action [Trivy Vulnerability Scanner](https://github.com/marketplace/actions/trivy-vulnerability-scanner).

Setting up Github Actions is easy. Go to your repo, click on Actions and then hit the new workflow button. You also want to skip any of the pre-defined setup for this if you're following along at home and just click on "set up a workflow yourself".

First, we're going to define the basics: triggers and checkout of the repo.

```yaml
name: Security Scan

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
```

Seems pretty simple so far right? And oddly familiar if you're worked with Azure DevOps pipelines as code.

This is the foundation of our action. In truth for the purpose of this particular example, we can probably get away without checking out the code, but lets leave it in there for future use in another blog post.

In the marketplace on the right of the screen, search for Trivy.

{% include figure image_path="/assets/images/posts/trivy-marketplace-action-github.png" alt="Searching for Trivy" caption="Searching for Trivy" %}

Use the action circled in red.

Adding an action in is as easy as copying and pasting in the YAML from the action. At time of writing, that looks like this:

```yaml
- name: Trivy Vulnerability Scanner
  uses: aquasecurity/trivy-action@0.0.7
  with:
    # image reference
    image-ref: 
    # exit code when vulnerabilities were found
    exit-code: # optional, default is 0
    # ignore unfixed vulnerabilities
    ignore-unfixed: # optional
    # severities of vulnerabilities to be displayed
    severity: # optional, default is UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
    # output format (table, json)
    format: # optional, default is table
```

As part of this workflow, I want to fail the build if there are any high or critical vulnerabilities, but I also still want to scan for the others. You may recall from the previous post that we can filter the severity but to do so we have to effectively run a scan twice. With this in mind, you will need to add the action twice.

```yaml
    - name: Trivy Scan - Unknown, Low and Medium Severity
      uses: aquasecurity/trivy-action@0.0.7
      with:
        # image reference
        image-ref: ruby:2.4.0
        # exit code when vulnerabilities were found
        exit-code: 0
        # severities of vulnerabilities to be displayed
        severity: UNKNOWN,LOW,MEDIUM
        
    - name: Trivy Scan - High and Critical Severity
      uses: aquasecurity/trivy-action@0.0.7
      with:
        # image reference
        image-ref: ruby:2.4.0
        # exit code when vulnerabilities were found
        exit-code: 1
        # severities of vulnerabilities to be displayed
        severity: HIGH,CRITICAL
```

As you can see in my example here, I've changed the exit code for both so that for unknown, low and medium results the task succeeds but any results for high and critical should fail the workflow.

I've also omitted the format and ignore-unfixed options for now.

Putting it all together, my workflow looks like this:

```yaml
name: Security Scan

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2

    - name: Trivy Scan - Unknown, Low and Medium Severity
      uses: aquasecurity/trivy-action@0.0.7
      with:
        # image reference
        image-ref: ruby:2.4.0
        # exit code when vulnerabilities were found
        exit-code: 0
        # severities of vulnerabilities to be displayed
        severity: UNKNOWN,LOW,MEDIUM
        
    - name: Trivy Scan - High and Critical Severity
      uses: aquasecurity/trivy-action@0.0.7
      with:
        # image reference
        image-ref: ruby:2.4.0
        # exit code when vulnerabilities were found
        exit-code: 1
        # severities of vulnerabilities to be displayed
        severity: HIGH,CRITICAL
```

Committing the file will trigger the workflow immediately based on the triggers specified in the file.

{% include figure image_path="/assets/images/posts/trivy-github-action-run.png" alt="Github action run of Trivy" caption="Github action run of Trivy" %}

As you can see in the image above, we have a passed and a failed action run.

Expanding the logs in the console gives us the test results.

{% include figure image_path="/assets/images/posts/trivy-github-action-result.png" alt="Test results" caption="Test results" %}

Trivy is still wowing me with it's speed, accuracy and overall ease of use. What is even better is that we can use the same tool across multiple platforms!

I'll be posting some follow up posts soon too on how to do this with a container that's built as part of the same workflow.