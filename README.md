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

run `Octant` from terminal

`Octant`




==========k9s ==========
snap install k9s  # if you already using ver. 20 of ubuntu as in my case snap comes as default

check by invoke k9s from cli, if fails check doing this export for config file:

`export KUBECONFIG=$HOME/.kube/config`

other issue is the permission on .k9s: permissions denied try this:

`mkdir ~/.k9s`

============================================

**Step 6:** Install [MetalLB](https://metallb.universe.tf/)
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/namespace.yaml

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/metallb.yaml

kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

```
**Step 7:** Find out the IP addresses being used by the nodes in the K8S cluster:

`kubectl get nodes -o wide`

You'll get output similar to the following:

```
NAME                 STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                     KERNEL-VERSION      CONTAINER-RUNTIME
kind-control-plane   Ready    master   5m23s   v1.19.1   172.19.0.4    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0
kind-worker          Ready    <none>   4m42s   v1.19.1   172.19.0.3    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0
kind-worker2         Ready    <none>   4m41s   v1.19.1   172.19.0.2    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0

```
**Step 8:** Find the [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) of the bridge into tthe cluster

`ip a s`

You'll get output similar to the following

```
.
.
.
4: br-956cdadd3d33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:8b:33:c3:fb brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/16 brd 172.19.255.255 scope global br-956cdadd3d33
       valid_lft forever preferred_lft forever
    inet6 fc00:f853:ccd:e793::1/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::42:8bff:fe33:c3fb/64 scope link
       valid_lft forever preferred_lft forever
    inet6 fe80::1/64 scope link
       valid_lft forever preferred_lft forever
.
.
.
```
**Step 12:** Create the ConfigMap for MetalLB using `vi` getting the range of addresses to include from the values provided in the field, `Usable range` from the output of the previous step.

`vi metallb-config.yaml`

Add the following content to `metallb-config.yaml`

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.19.255.1-172.19.255.250

```

**WHERE**

`172.19.255.1-172.19.255.250` are the IP addreses determined from the steps above.

**Step 13:** Apply the ConfigMap to the K8S cluster

`kubectl apply -f metallb-config.yaml`

**Step 14:** Create a test K8S deployment and bind it to a service
```
kubectl create deploy nginx --image nginx

kubectl expose deploy nginx --port 80 --type LoadBalancer

```

**Step 15:** Confirm that the `nginx` service is bound to an external IP address

`kubectl get services`

You'll get out put similar to the following:

```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1       <none>         443/TCP        11m
nginx        LoadBalancer   10.96.148.110   172.19.255.1   80:32326/TCP   9s
```
Notice that the nginx service publishes an `EXTERNAL-IP`.

**Step 16:** Run the `curl` against the `EXTERNAL-IP`, for example:

`curl 172.19.255.1`

You'll get output similar to the following:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
