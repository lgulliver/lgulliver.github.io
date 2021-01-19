---
author: Liam Gulliver
title: Scanning Terraform, Kubernetes and More for Policy Compliance with Terrascan
tags: devops devsecops terrascan accurics kubernetes
date: 2021-01-11 14:30:00
---

I was recently introduced a new security and compliance scanning tool called [Terrascan](https://github.com/accurics/terrascan). It's another free and open source tool, just like another tool I've covered previously in this space called [Trivy](https://lgulliver.github.io/container-security-scanning-with-trivy-in-azure-devops/).

From the brief look I've had into Terrascan (a deeper dive to come!), it allows us to automate the compliance and security scans against a [pre-defined set of policies](https://docs.accurics.com/projects/accurics-terrascan/en/latest/policies/) or custom policies as part of the CI process.

It has support for Terraform, Azure, GCP, AWS, Kubernetes (manifests, Helm, Kustomize), though as it doesn't seem to have support for Dockerfiles, it's a tool to be used alongside something like [Trivy](https://lgulliver.github.io/container-security-scanning-with-trivy-in-azure-devops/).

For my quick test, I installed Terrascan in my Ubuntu 20.04 WSL image on my machine (though you can also [use it as a Docker image](https://docs.accurics.com/projects/accurics-terrascan/en/latest/getting-started/usage/#using-docker)) and pulled down a repo with a really basic Kubernetes application in, which I know also has some intentional mistakes/omissions in.

Installation:

```bash
$ curl --location https://github.com/accurics/terrascan/releases/download/v1.2.0/terrascan_1.2.0_Linux_x86_64.tar.gz --output terrascan.tar.gz
$ tar -xvf terrascan.tar.gz
  x CHANGELOG.md
  x LICENSE
  x README.md
  x terrascan
$ install terrascan /usr/local/bin
```

To confirm Terrascan is installed, simply run the command `terrascan` in the terminal. You should receive something like the below back:

```bash
$ terrascan
Terrascan

Detect compliance and security violations across Infrastructure as Code to mitigate risk before provisioning cloud native infrastructure.
For more information, please visit https://docs.accurics.com

Usage:
  terrascan [command]

Available Commands:
  help        Help about any command
  init        Initialize Terrascan
  scan        Detect compliance and security violations across Infrastructure as Code.
  server      Run Terrascan as an API server
  version     Terrascan version

Flags:
  -c, --config-path string   config file path
  -h, --help                 help for terrascan
  -l, --log-level string     log level (debug, info, warn, error, panic, fatal) (default "info")
  -x, --log-type string      log output type (console, json) (default "console")
  -o, --output string        output type (json, yaml, xml) (default "yaml")
```

Then to scan my Kubernetes manifests, all I need to do is navigate to the directory they're sitting in and run the following command to tell Terrascan I'm scanning Kubernetes manifests (default is Terraform):

```bash
terrascan scan -i k8s
```

By default, the output is YAML and out to screen rather than to a file. In any case, the scan is incredibly quick (my initial test ran in less than a second) and produced the following:

```yaml
results:
    violations:
        - rule_name: containerResourcesNotDefined
          description: Container does not have resource limitations defined
          rule_id: accurics.kubernetes.IAM.107
          severity: MEDIUM
          category: Identity and Access Management
          resource_name: carsunlimited-cart-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/cart-deployment.yaml
          line: 1
        - rule_name: containerResourcesNotDefined
          description: Container does not have resource limitations defined
          rule_id: accurics.kubernetes.IAM.107
          severity: MEDIUM
          category: Identity and Access Management
          resource_name: carsunlimited-inventory-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/inventory-deployment.yaml
          line: 1
        - rule_name: containerResourcesNotDefined
          description: Container does not have resource limitations defined
          rule_id: accurics.kubernetes.IAM.107
          severity: MEDIUM
          category: Identity and Access Management
          resource_name: carsunlimited-purchase-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/purchase-deployment.yaml
          line: 1
        - rule_name: containerResourcesNotDefined
          description: Container does not have resource limitations defined
          rule_id: accurics.kubernetes.IAM.107
          severity: MEDIUM
          category: Identity and Access Management
          resource_name: carsunlimited-web-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/web-deployment.yaml
          line: 1
        - rule_name: defaultNamespaceUsed2
          description: The default namespace should not be used
          rule_id: accurics.kubernetes.OPS.461
          severity: LOW
          category: Operational Efficiency
          resource_name: carsunlimited-cart-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/cart-deployment.yaml
          line: 1
        - rule_name: defaultNamespaceUsed2
          description: The default namespace should not be used
          rule_id: accurics.kubernetes.OPS.461
          severity: LOW
          category: Operational Efficiency
          resource_name: carsunlimited-inventory-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/inventory-deployment.yaml
          line: 1
        - rule_name: defaultNamespaceUsed2
          description: The default namespace should not be used
          rule_id: accurics.kubernetes.OPS.461
          severity: LOW
          category: Operational Efficiency
          resource_name: carsunlimited-purchase-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/purchase-deployment.yaml
          line: 1
        - rule_name: defaultNamespaceUsed2
          description: The default namespace should not be used
          rule_id: accurics.kubernetes.OPS.461
          severity: LOW
          category: Operational Efficiency
          resource_name: carsunlimited-web-deployment
          resource_type: kubernetes_deployment
          file: /mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/web-deployment.yaml
          line: 1
    count:
        low: 4
        medium: 4
        high: 0
        total: 8
```

The really cool thing here is that the policies seem to also be categorised against the [Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/).

The output can also be provided as JSON or XML - you may recall if you read the [Trivy posts I wrote last year that Azure DevOps needs the output in XML](https://lgulliver.github.io/trivy-scan-results-to-azure-devops/). 

To output it to XML, you need to append the `-o` or `--output` option with the value `xml`:

```bash
terrascan scan -i k8s -o xml
```

This will give you XML output ~~that should be compatible with the JUnit XML format~~:

**UPDATE: No it isn't compatible with JUnit/XUnit/NUnit or any other format supported by Azure DevOps**

```xml
<results>
  <violations>
    <violation rule_name="defaultNamespaceUsed2" description="The default namespace should not be used" rule_id="accurics.kubernetes.OPS.461" severity="LOW" category="Operational Efficiency" resource_name="carsunlimited-cart-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/cart-deployment.yaml" line="1"></violation>
    <violation rule_name="defaultNamespaceUsed2" description="The default namespace should not be used" rule_id="accurics.kubernetes.OPS.461" severity="LOW" category="Operational Efficiency" resource_name="carsunlimited-inventory-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/inventory-deployment.yaml" line="1"></violation>
    <violation rule_name="defaultNamespaceUsed2" description="The default namespace should not be used" rule_id="accurics.kubernetes.OPS.461" severity="LOW" category="Operational Efficiency" resource_name="carsunlimited-purchase-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/purchase-deployment.yaml" line="1"></violation>
    <violation rule_name="defaultNamespaceUsed2" description="The default namespace should not be used" rule_id="accurics.kubernetes.OPS.461" severity="LOW" category="Operational Efficiency" resource_name="carsunlimited-web-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/web-deployment.yaml" line="1"></violation>
    <violation rule_name="containerResourcesNotDefined" description="Container does not have resource limitations defined" rule_id="accurics.kubernetes.IAM.107" severity="MEDIUM" category="Identity and Access Management" resource_name="carsunlimited-cart-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/cart-deployment.yaml" line="1"></violation>
    <violation rule_name="containerResourcesNotDefined" description="Container does not have resource limitations defined" rule_id="accurics.kubernetes.IAM.107" severity="MEDIUM" category="Identity and Access Management" resource_name="carsunlimited-inventory-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/inventory-deployment.yaml" line="1"></violation>
    <violation rule_name="containerResourcesNotDefined" description="Container does not have resource limitations defined" rule_id="accurics.kubernetes.IAM.107" severity="MEDIUM" category="Identity and Access Management" resource_name="carsunlimited-purchase-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/purchase-deployment.yaml" line="1"></violation>
    <violation rule_name="containerResourcesNotDefined" description="Container does not have resource limitations defined" rule_id="accurics.kubernetes.IAM.107" severity="MEDIUM" category="Identity and Access Management" resource_name="carsunlimited-web-deployment" resource_type="kubernetes_deployment" file="/mnt/c/Checkout/Internal/CarsUnlimited-Kubernetes/02-carsunlimited/web-deployment.yaml" line="1"></violation>
  </violations>
  <count low="4" medium="4" high="0" total="8"></count>
</results>
```

To save it as an XML file, all you need to do is append `> result.xml`. The complete command looks as follows:

```bash
terrascan scan -i k8s -o xml > result.xml
```

In a follow up post, I'll cover integrating this into the CI/CD pipeline in Azure DevOps so that you can fail builds on Terrascan failures.