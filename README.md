# Overview

This is not intended to repalce the current documention for using Concourse with Helm.  The problem though is that Kubernetes gives end users many choices in order to access cluster resources from the outside, and the docs cannot cover all of that.  By using some of the examples and ideas in this repository, the hope is that it will help users find out the best way to get things set up for each customized Kubernetes cluster.

For the most part, please follow the main instructions located here: 
[Concourse Install with Helm](https://docs.pivotal.io/p-concourse/v5/installation/install-concourse-helm)

What is included in this repo are some example scripts and deployment files to start with.  These files can control just about anything that Concourse can be configured with More values and options are located here:
[Deployment Values YAML Structure](https://github.com/helm/charts/blob/master/stable/concourse/values.yaml)

## Choices

While following the documentation, several choices must be made.  There likely is no wrong way to do it, but knowing that all of configuration is available to add to the helm chart customization file is import.  Some things to consider:

#### Private / Public Repo 

If using a public Docker Hub or any other public respoitory, the steps around setting up a registry credential and configuring tiller and the chart to use it can be omitted.  This can also be tested before running the helm chart apply command.

#### Storage Class

The docs mention setting up a Kubernetes ```storageclass```, however on some clusters, there will already be a default storage class, which could cause a conflict if another default is added.  GCP for one, already has it defined, so that section in the docs can be skipped.

#### Resource Sizing
This refers to the ability to change things like worker count, persistent disk size, and affinity.  Through trial and error, these can be adjusted and re-applied to the helm deployment.

#### SSL 

Likely if a Concourse instance were to be opened to a large user base, we would want to use an Ingress Controller and attach and SSL Certificate to it, so that all traffic comes through encrypted.  There are a couple of examples in this repository that set up SSL using cert-manager.

## Deployment Scenarios

There are a couple of generic things that should be applicable in all cases.  Namely, Tiller must be configured on your Kubernetes cluster in order to use the chart.  **As of Concourse 5.5.7, only Helm v2 is supported.** The Helm documentation points to Helm v3.

## Available Scenarios
* Vanilla Deployment
* Use of Docker Hub or Harbor for images
* Metal-LB as Load Balancer
* Using NGINX Ingress Controller
* Using GKE and GCE Ingress

## Other Configurations

* cert-manager with automatic SSL cert for web frontend
* Connection to existing Credhub (for use with Platform Automation)
