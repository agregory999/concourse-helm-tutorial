# Deployment Options

Each file should be named such that it is easy to see where it was originally deployed. Anything that uses SSL required the presence of a secret containing the TLS information.  For the purposes of this repository, [cert-manager](https://cert-manager.io/docs/installation/) was installed and configured on the cluster prior to applying the chart.  There is a separate doc on that [over here](../cert-manager/README.md)

## Vanilla Installation

What you get from the [Concourse docs](https://docs.pivotal.io/p-concourse/v5/installation/install-concourse-helm/) gives your the following.  Applying this gives you a basic Concourse install with no external access.  Here is the example given, without the pullSecret, which isn't required if using Docker Hub public repositories:

```
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
NOTE 1: The worker disk size is adjusted as well, as I quickly found that to be insufficient. 
NOTE 2: The externalUrl must be set to something that resolves back to the IP that the LoadBalancer gives back.  In this case, Google Cloud DNS was used.  **If the externalUrl is not set, the Concourse login does not work.**

```
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


