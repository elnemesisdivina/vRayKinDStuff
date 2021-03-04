# vRayKinDStuff
This repo will try to explain the step zero towards the practice of K8s on your own mac :) folks with Linux distro can follow as jsut some commands need to be adapted. all of this based on this list:

K8s clusters
[KinD](https://kind.sigs.k8s.io/)

CNI
[Antrea](https://antrea.io/)

L4 LB
[MetalLB Load Balancer](https://metallb.universe.tf/)

K8s Management tool kit
[Octant](https://reference.octant.dev/?path=/docs/docs-intro--page#getting-started)

k8s apps manager
[Helm](https://helm.sh/)
[Kubeapps](https://kubeapps.com/)

curated Apps catalog (Bitnami)
[Tanzu Application Catalog](https://bitnami.com/)

Observability with steroids!
[Tanzu Observability](https://docs.wavefront.com/)

Policy enforcement adn K8s cluster managment:
[Tanzu Mission Control](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/index.html)

Continous Delivery 
[ArgoCD](https://argoproj.github.io/argo-cd/)

# Work in progress
Continous Delivery
[Concurse](https://concourse-ci.org/)

Container Registry
[Harbor](https://goharbor.io/docs/2.2.0/install-config/)


## Step zero the is not a special sauce for this so assuming you already Install `kubectl`

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

``` 

 - Docker
 - Kind
 - Kubectl
 -

## Setting Up Kind on your Mac

**Step 1:** 


