# vRayKinDStuff
This repo will try to explain the step zero towards the practice of K8s on your own mac :) folks with Linux distro can follow as jsut some commands need to be adapted. all of this based on this list:

##### K8s clusters on Docker Containers
-[KinD](https://kind.sigs.k8s.io/)
##### CNI
-[Antrea](https://antrea.io/)
##### L4 LB
-[MetalLB Load Balancer](https://metallb.universe.tf/)
##### K8s Management tool kit
-[Octant](https://reference.octant.dev/?path=/docs/docs-intro--page#getting-started)
#### k8s apps manager
-[Helm](https://helm.sh/)
-[Kubeapps](https://kubeapps.com/)
##### curated Apps catalog (Bitnami)
-[Tanzu Application Catalog](https://bitnami.com/)
##### Observability with steroids!
-[Tanzu Observability](https://docs.wavefront.com/)
##### Policy enforcement adn K8s cluster managment:
-[Tanzu Mission Control](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/index.html)
##### Continous Delivery 
-[ArgoCD](https://argoproj.github.io/argo-cd/)
###### Work in progress...
##### Continous Delivery
-[Concourse](https://concourse-ci.org/)
##### Container Registry
-[Harbor](https://goharbor.io/docs/2.2.0/install-config/)

## Setting Up Kind on your Mac

**Step 0:** is to install `docker` on your mac
`https://docs.docker.com/docker-for-mac/install/`
 ##### Check if this fine (Client & Server)
```
docker version
```

**Step 1:** the is not a special sauce just install `kubectl`

```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

``` 

 - Docker
 - Kind
 - Kubectl
 -
**Step 3:** Check to make sure kubectl is running

`kubectl version --client --short`

ourput example:
```
Client Version: v1.20.0
```
**Step 4:** Install [KinD](https://kind.sigs.k8s.io/)
```
brew install kind
```
or 
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-darwin-amd64

```

**Step 4:** Create your config file using `vim` this will be use for `KinD` on how to crete your K8s cluster

`vim kind-antrea.yaml`

Edit and add your setting, in my case I decide to add `Antrea` CNI from the beginning, `kind-config.yaml`

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4 #always check api version
networking:
  disableDefaultCNI: true #disable kindet 
  podSubnet: 192.168.0.0/16
nodes: #set 1 master and 3 worker nodes
- role: control-plane
- role: worker
- role: worker
- role: worker
```
**Step 4:** Create your `KinD` cluster

`kind create cluster --name ernesto --config antrea-config.yaml`

Output fo this command example:
```
Creating cluster "ernesto" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼 
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-ernesto"
You can now use your cluster with:

kubectl cluster-info --context kind-ernesto

Thanks for using kind! 😊
```

run to check your recent create Docker cntrs aka Kubernetes Nodes:

`docker ps`

```add name ernesto output
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                       NAMES
b7e28a6d0493        kindest/node:v1.19.1   "/usr/local/bin/entr…"   About a minute ago   Up About a minute                               ernesto-worker
...
2a1d1484f828        kindest/node:v1.19.1   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   127.0.0.1:45109->6443/tcp   ernesto-control-plane
```

`kind get clusters`
add output of ernesto

`kubectl config view`

#Export kubeconfig of new created cluster on kind

`export KUBECONFIG="$(kind get kubeconfig --name="ernesto")"`


`kubectl config view`

`kubectl get nodes`

`kubectl get pods -A`


Just in case you need it this is the way to delete the cluster 

`kind delete cluster --name ernesto`

**Step 4:** Config `Antrea` control plane and Dataplane (Antrea CNI for K8s)

Check also `https://antrea.io/docs/v0.11.1/kind/` for specifics on how to deploy antrea CNI on `KinD` 
TL;DR: "the OVS netdev datapath dopes not support TX checksumm offloading here is the link ref. if you like to goo deeper `https://github.com/vmware-tanzu/antrea/issues/14`" 

cloning the `Antrea` Repo

`git clone https://github.com/vmware-tanzu/antrea.git`

exceute teh fix to the nodes:

`kind get nodes --name antreakind | xargs ~/antrea/hack/kind-fix-networking.sh`

you acan apply the following manisfesto just TAG check release you wan in my case `<TAG>` is v0.12.0 aka TAG=`v0.12.0`

`kubectl apply -f https://github.com/vmware-tanzu/antrea/releases/download/<TAG>/antrea-kind.yml`

**Step 5:** Install `Octant` and install  the `Antrea` `Octant` plugin

Install Octant on Mac

`brew install octant`

Clone Octant git repo,

`git clone https://github.com/vmware-tanzu/octant.git`

Build an `Octant` test plugin `https://reference.octant.dev/?path=/docs/docs-plugins-2-go-plugins--page`

`cd octant`
`go run build.go install-test-plugin`

apply from recent clone `Antrea` repo as well, this will be useful for the `Antrea` `Octant` Plugin or get from this link

`https://github.com/vmware-tanzu/antrea/releases/download/v0.12.2/antrea-octant-plugin-darwin-x86_64`

so then move to the default octant plpugin folder:

`mv antrea-octant-plugin-darwin-x86_64 ~/.config/octant/plugins/`

export kubeconf

`export KUBECONFIG=~/.kube/config`

run `Octant`
`Octant`






==========k9s==========
snap install k9s  # if you already using ver. 20 of ubuntu as in my case snap comes as default

check by invoke k9s from cli, if fails check doing this export for config file:

`export KUBECONFIG=$HOME/.kube/config`

other issue is the permission on .k9s: permissions denied try this:

`mkdir ~/.k9s`

=================
