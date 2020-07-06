---
author: Liam Gulliver
title: Container Security Scanning with Trivy and Azure DevOps
tags: docker security scanning azure-devops devops trivy containers
date: 2020-07-06 07:49:00
---

Recently I've been taking a deeper look into how we can bake security scanning and practices into CI/CD pipelines without the price tag security tooling tends to be. I also wanted it to be integrated into my pipelines and have it easy to set up and run.

I've looked at tools such as Anchore, which is great and I'll have a blog post on that soon too, but there was something I couldn't quite put my finger on that left me wanting more.

Continuing to explore my options led me to [Trivy](https://github.com/aquasecurity/trivy) by Aqua Security.

Its free, open source and most importantly, its pretty fast too.

There's currently no marketplace extension for Trivy with Azure DevOps, but fear not, Trivy is pretty easy to get started with. Side note: it's also really easy to use in GitHub Actions as well.

You can do a lot with it too from scanning OCIs, baking it in as part of your `Dockerfile` through to scanning file systems. For the purpose of this blog post though, we're going to keep it simple and scan images that have been built already and pushed up to Docker Hub.

If this is too long to read and you want to skip straight to the good stuff, I've got a repo here: https://github.com/lgulliver/ContainerScanning

## Getting started

Getting up and running is pretty simple. As a preference, I tend to use Microsoft hosted agents and running there is really straightforward.

Using the latest Ubuntu-based agents, we're going to download the latest `.deb` package and install it. We're also going to output the version to test Trivy got installed correctly.

When using the MS hosted agents, we're going to need to do this every time because they're disposable. For me, I'm finding this takes ~30 seconds at most. The scan is shorter!

```yaml
name: $(SourceBranchName)-$(Date:yyyyMMdd)$(Rev:.r)

trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

variables:
  trivyVersion: 0.9.1

steps:
- script: |
    sudo apt-get install rpm
    wget https://github.com/aquasecurity/trivy/releases/download/v$(trivyVersion)/trivy_$(trivyVersion)_Linux-64bit.deb
    sudo dpkg -i trivy_$(trivyVersion)_Linux-64bit.deb
    trivy -v
  displayName: 'Download and install Trivy'
```

## Scanning containers

I'm going to scan a pre-existing container from Docker Hub. This the command line task.

```yaml
- task: CmdLine@2
  displayName: "Run trivy scan"
  inputs:
    script: |
      trivy image liamgu/azuredevopscontainersdemo:74
```

For me, this scan took all of 5 seconds to run against my demo container `liamgu/azuredevopscontainersdemo:74` image and the results were great. It scans the image itself and whatever is on the image. This particular image is based on `httpd:latest`.

{% include figure image_path="/assets/images/posts/trivy-scan-result.png" alt="Trivy scan result example" caption="Snippet of the Trivy scan" %}

The above image is just a snippet of the scan, you can go and see the full results at [https://dev.azure.com/lgulliver/ContainerScanning/_build/results?buildId=146&view=logs&j=12f1170f-54f2-53f3-20dd-22fc7dff55f9&t=5caf77c8-9b10-50ef-b5c7-ca89c63e1c86](https://dev.azure.com/lgulliver/ContainerScanning/_build/results?buildId=146&view=logs&j=12f1170f-54f2-53f3-20dd-22fc7dff55f9&t=5caf77c8-9b10-50ef-b5c7-ca89c63e1c86) if you really want to. 

It gives us what library the vulnerability is in, the ID of the vulnerability, severity, what version of the library is installed, what version it's fixed in.

Though at this stage, our build will succeed regardless of the result which is not ideal.

We can even choose to ignore certain vulnerabilities if we want to for whatever reason, which might be accepting risk or it not having impact on your particular use case. This can be done with a `.trivyignore` file that might look something like this:

```
# Accept the risk
CVE-2018-14618

# No impact in our settings
CVE-2019-1543
```

## Failing the build based on findings

Its pretty straightforward to make the task fail and even customise the rules around failure.

Trivy supports setting exit codes and filters for when its run.

```yaml
- task: CmdLine@2
  displayName: "Run trivy scan"
  inputs:
    script: |
      trivy image --exit-code 0 --severity LOW,MEDIUM liamgu/azuredevopscontainersdemo:74
      trivy image --exit-code 1 --severity HIGH,CRITICAL liamgu/azuredevopscontainersdemo:74
```

Doing this means that when it finds any high or critical results, it'll fail the build and you can act accordingly. The downside to this of course is that it actually runs the scan twice, but filters the results.

{% include figure image_path="/assets/images/posts/trivy-scan-result-fail.png" alt="Trivy scan result example" caption="Snippet of the Trivy scan failed task" %}

You can also output the results to a format such as JUnit, but at time of writing, there is a bug with the formatting resulting in an invalid report meaning Azure DevOps can't import it. A fix has been submitted though and is currently being tested, so I expect it'll be resolved in the next release and I shall follow up with how to integrate the results into Azure DevOps test reporting in due course.