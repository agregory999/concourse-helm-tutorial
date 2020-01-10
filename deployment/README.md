# Deployment Values

In order to affect the final state of the deployment, we make changes to the **deployment-values.yaml** file.  This can be done in one of 2 ways.  First, we can take all changes want, put them into the file in the correct order, and install/upgrade the Helm chart.  Or, we can beak down the types of things we want into modular sections, and apply them with multiple switches in the command line.  

Example:

```
agregory@jump:~/concourse-helm-tutorial/deployment$ helm upgrade --install concourse-helm \
--values deployment-values-lb-gce.yaml ${UNZIPPED_DOWNLOAD}/charts/
```
or
```bash
andrew@jump:~/concourse-helm-tutorial/deployment$ helm upgrade --install concourse-helm \
-f modular/concourse-web-external-url.yaml -f modular/worker-affinity.yaml -f modular/image-harbor.yaml \
-f modular/web-ingress-nginx-no-ssl.yaml  -f modular/external-credhub.yaml ${UNZIPPED_DOWNLOAD}/charts/
```
*where UNZIPPED_DOWNLOAD refers to the directory on your system where the artifact from PivNet was extracted.  The chsrts/ directory should be in there*

# Practical Examples (non-modular)

Each file should be named such that it is easy to see where it was originally deployed. Anything that uses SSL required the presence of a secret containing the TLS information.  For the purposes of this repository, [cert-manager](https://cert-manager.io/docs/installation/) was installed and configured on the cluster prior to applying the chart.  There is a separate doc on that [over here](../cert-manager/README.md)


## Vanilla Installation

What you get from the [Concourse docs](https://docs.pivotal.io/p-concourse/v5/installation/install-concourse-helm/) gives your the following.  Applying this gives you a basic Concourse install with no external access.  Here is the example given, without the pullSecret, which isn't required if using Docker Hub public repositories:

```yaml
---
image: agregory99/concourse
imageTag: 5.5.6-ubuntu
postgresql:
  image:
    registry:
    repository: agregory99/postgres
    tag: 0.0.1
```

Apply this to see the simplest installation.  You will be required to follow the NOTE section to gain access from a browser:
>  * From outside the cluster, run these commands in the same shell:
>
>    export POD_NAME=$(kubectl get pods --namespace default -l "app=concourse-helm-web" -o jsonpath="{.items[0].metadata.name}")
>    echo "Visit http://127.0.0.1:8080 to use Concourse"
>    kubectl port-forward --namespace default $POD_NAME 8080:8080

So...not very practical

## Add a LoadBalancer (Clouds that support it)

On GCP or other clouds where the creation of a Kubernetes LoadBalancer triggers the automatic creation of an infrastructure load balancer, the adjustment to the deployment file is shown here.  

-NOTE 1: The worker disk size is adjusted as well, as I quickly found that to be insufficient. 
-NOTE 2: The externalUrl must be set to something that resolves back to the IP that the LoadBalancer gives back.  In this case, Google Cloud DNS was used.  **If the externalUrl is not set, the Concourse login does not work.**

```yaml
---
image: agregory99/concourse
imageTag: 5.5.6-ubuntu
concourse:
  web:
    externalUrl: http://conc.gke.arg-pivotal.com:8080
web:
  service:
    type: LoadBalancer
worker:
  replicas: 3
postgresql:
  image:
    registry:
    repository: agregory99/postgres
    tag: 0.0.1
persistence:
  worker:
    size: 100Gi
```

## Using MetalLB as a LoadBalancer -AND- Harbor for Image Registry

On some platforms, such as PKS on vSphere, there is no assigned IP for a service of type LoadBalancer.  This means that a LoadBalancer is useless for external access.  To get around that, [MetalLB](https://metallb.universe.tf/) can easily be installed prior to application of the helm chart for Concourse.  What this does is create a pool of usable external IP Addresses which then are assigned when using the YAML from above.  Since this IP address is very likely internal, it can simply be referenced directly by the **externalUrl** field.

In this example, since this is running on vSphere, and since Harbor Registry is installed, we can use both:

```yaml
---
image: harbor.homelab.arg-pivotal.com/concourse-helm/concourse
imageTag: 5.5.6-ubuntu
imagePullSecrets: ["regcred"] #remove this line for public image registry
concourse:
  web:
    externalUrl: http://192.168.1.241:8080
web:
  service:
    type: LoadBalancer
postgresql:
  image:
    registry: harbor.homelab.arg-pivotal.com
    repository: concourse-pks-helm/postgres
    tag: 0.0.1
    pullSecrets: ["regcred"] # remove for public image registry
```
## NGINX Ingress Controller 

Using an Ingress Controller is likely the most flexible way to provide access to Kubernetes resources.  Ingress allows for multiple mappings, SSL termination, and many other features.  Many clouds have their own Ingress implementations, but [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) is a top choice.  The deployment-values file can be updated to allow for external access with or without SSL.  

In order to get this working, the ingress controller needs to be installed per the docs.  Once that is done, a **LoadBalancer** is created within the Kubernetes cluster, which then triggers (on some clouds) an Infrastructure Load Balancer, and if Metal LB was installed, a usable external IP.  Either way, the ingress controller relies on a wildcard DNS entry to map all subdomains to the same entry point.  

In the example below, \*.concourse-pks-helm.pks.gcp-nonprod.arg-pivotal.com is mapped to the EXTERNAL IP that the nginx-ingress was given, in this case by Metal LB.  

-NOTE: Annotations need to be added 

```yaml
---
image: agregory99/concourse
imageTag: 5.5.6-ubuntu
concourse:
  web:
    externalUrl: https://web.concourse-pks-helm.pks.gcp-nonprod.arg-pivotal.com
web:
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: "nginx"
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
    hosts:
      - web.concourse-pks-helm.pks.gcp-nonprod.arg-pivotal.com
    tls:
    - hosts:
      - web.concourse-pks-helm.pks.gcp-nonprod.arg-pivotal.com
      secretName: concourse-external-cert
worker:
  replicas: 3
postgresql:
  image:
    registry:
    repository: agregory99/postgres
    tag: 0.0.1
persistence:
  worker:
    size: 100Gi
```
## GKE Installation with GKE Ingress (No SSL)

As mentioned, this option will create a deployment that uses Ingress to provide access to Concourse.  GKE will sense the creation of ingress and create a GCP Load Balancer.  An annotation is used to tell GCP the named static IP to take for the 

## GKE Installation with GKE Ingress and SSL

As mentioned, this option will create a deployment that uses Ingress to provide access to Concourse.  GKE will sense the creation of ingress and create a GCP Load Balancer.  *cert-manager* will also intercept the ingress creation and use either DNS or HTTP01 (depending on configuration) challenges to generate a real certificate.  Upon issuance, the secret is updated, the ingress is updated, and finally GCP will update the Load Balancer with the certificate information.  Nice!

To see this in action:
1) A ClusterIssuer for Let's Encrypt called ```letsencrypt-prod``` was created
2) The hostname for ```externalUrl``` and ```hosts``` was defined, created in DNS, and pointed to the named External IP called ```conc-helm-gke-web```
3) The concourse and postgres charts were extracted from the pivnet-cli download and pushed to Docker Hub

### Command

```helm upgrade --install concourse-helm --values deployment-values-gke-ingress-ssl-only.yaml ./charts```

### deployment-values-gke-ingress-ssl-only.yaml
```yaml
---
image: agregory99/concourse
imageTag: 5.5.6-ubuntu
concourse:
  web:
    externalUrl: https://conc.gke.arg-pivotal.com
web:
  service:
    type: NodePort
  ingress:
    enabled: true
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      kubernetes.io/ingress.global-static-ip-name: conc-helm-gke-web
      kubernetes.io/ingress.allow-http: "false"
    hosts:
      - conc.gke.arg-pivotal.com
    tls:
      - hosts:
        - conc.gke.arg-pivotal.com
        secretName: concourse-external-cert
worker:
  replicas: 3
postgresql:
  image:
    registry:
    repository: agregory99/postgres
    tag: 0.0.1
persistence:
  worker:
    size: 100Gi
```

## Connect Concourse to External CredHub

Conenct to external
