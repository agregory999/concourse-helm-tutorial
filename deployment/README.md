# Deployment Options

Each file should be named such that it is easy to see where it was originally deployed. Anything that uses SSL required the presence of a secret containing the TLS information.  For the purposes of this repository, [cert-manager](https://cert-manager.io/docs/installation/) was installed and configured on the cluster prior to applying the chart.  There is a separate doc on that [over here](../cert-manager/README.md)

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


