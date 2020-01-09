# concourse-helm-tutorial


 ▄████▄   ▒█████   ███▄    █  ▄████▄   ▒█████   █    ██  ██▀███    ██████ ▓█████
▒██▀ ▀█  ▒██▒  ██▒ ██ ▀█   █ ▒██▀ ▀█  ▒██▒  ██▒ ██  ▓██▒▓██ ▒ ██▒▒██    ▒ ▓█   ▀
▒▓█    ▄ ▒██░  ██▒▓██  ▀█ ██▒▒▓█    ▄ ▒██░  ██▒▓██  ▒██░▓██ ░▄█ ▒░ ▓██▄   ▒███
▒▓▓▄ ▄██▒▒██   ██░▓██▒  ▐▌██▒▒▓▓▄ ▄██▒▒██   ██░▓▓█  ░██░▒██▀▀█▄    ▒   ██▒▒▓█  ▄
▒ ▓███▀ ░░ ████▓▒░▒██░   ▓██░▒ ▓███▀ ░░ ████▓▒░▒▒█████▓ ░██▓ ▒██▒▒██████▒▒░▒████▒
░ ░▒ ▒  ░░ ▒░▒░▒░ ░ ▒░   ▒ ▒ ░ ░▒ ▒  ░░ ▒░▒░▒░ ░▒▓▒ ▒ ▒ ░ ▒▓ ░▒▓░▒ ▒▓▒ ▒ ░░░ ▒░ ░
  ░  ▒     ░ ▒ ▒░ ░ ░░   ░ ▒░  ░  ▒     ░ ▒ ▒░ ░░▒░ ░ ░   ░▒ ░ ▒░░ ░▒  ░ ░ ░ ░  ░
░        ░ ░ ░ ▒     ░   ░ ░ ░        ░ ░ ░ ▒   ░░░ ░ ░   ░░   ░ ░  ░  ░     ░
░ ░          ░ ░           ░ ░ ░          ░ ░     ░        ░           ░     ░  ░
░                            ░

# Basic instructions

For the most part, please follow the main instructions located here: 
https://docs.pivotal.io/p-concourse/v5/installation/install-concourse-helm/

What is included in this repo are some example scripts and deployment files to start with.  More values and options are located here:
https://github.com/helm/charts/blob/master/stable/concourse/values.yaml

# Deployment Scenarios

There are a couple of generic things that should be applicable in all cases.  Namely, Tiller must be configured on your Kubernetes cluster in order to use the chart.

## Available Scenarios
* Vanilla Deployment
* Use of Docker Hub or Harbor for images
* Metal-LB as Load Balancer
* Using NGINX Ingress Controller
* Using GKE and GCE Ingress

## Other Configurations

* cert-manager with automatic SSL cert for web frontend
* Connection to existing Credhub (for use with Platform Automation)
