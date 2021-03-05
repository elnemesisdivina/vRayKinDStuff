# vRayKinDStuff
This repo will try to explain the step zero towards the practice of K8s on your own mac :) folks with Linux distro can follow as just some commands need to be adapted. all of this based on this list:

##### K8s clusters on Docker Containers
-[KinD](https://kind.sigs.k8s.io/)
##### CNI
-[Antrea](https://antrea.io/)
##### L4 LB
-[MetalLB](https://metallb.universe.tf/)
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
-[Tanzu Service Mesh]

## Setting Up Kind on your Mac

**Step 0:** is to install `docker` on your mac

`https://docs.docker.com/docker-for-mac/install/`

##### Check if this fine (Client & Server)

```
docker version
```

**Step 1:** the is not a special sauce just install `kubectl`

```shell
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

```shell
Client Version: v1.20.0
```
**Step 4:** Install [KinD](https://kind.sigs.k8s.io/)

```shell
brew install kind
```
or 

```shell
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

```shell
add name ernesto output
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                       NAMES
b7e28a6d0493        kindest/node:v1.19.1   "/usr/local/bin/entr…"   About a minute ago   Up About a minute                               ernesto-worker
...
2a1d1484f828        kindest/node:v1.19.1   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   127.0.0.1:45109->6443/tcp   ernesto-control-plane
```

`kind get clusters`

output example:

```
ernesto
```

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

excecute teh fix to the nodes:

`kind get nodes --name antreakind | xargs ~/antrea/hack/kind-fix-networking.sh`

output sample:

```shell
kind get nodes --name antreakind | xargs ./kind-fix-networking.sh 
Disabling TX checksum offload for node antreakind-worker (veth9016bb5)
Actual changes:
tx-checksumming: off
	tx-checksum-ip-generic: off
	tx-checksum-sctp: off
tcp-segmentation-offload: off
	tx-tcp-segmentation: off [requested on]
	tx-tcp-ecn-segmentation: off [requested on]
	tx-tcp-mangleid-segmentation: off [requested on]
	tx-tcp6-segmentation: off [requested on]
net.ipv4.tcp_retries2 = 4
Disabling TX checksum offload for node antreakind-control-plane (veth3af0c96)
Actual changes:
tx-checksumming: off
	tx-checksum-ip-generic: off
	tx-checksum-sctp: off
tcp-segmentation-offload: off
	tx-tcp-segmentation: off [requested on]
	tx-tcp-ecn-segmentation: off [requested on]
	tx-tcp-mangleid-segmentation: off [requested on]
	tx-tcp6-segmentation: off [requested on]
net.ipv4.tcp_retries2 = 4
```

you acan apply the following manisfesto just TAG check release you wan in my case `<TAG>` is v0.12.0 aka TAG=`v0.12.0`

`kubectl apply -f https://github.com/vmware-tanzu/antrea/releases/download/v0.12.0/antrea-kind.yml`

output sample:

```shell
customresourcedefinition.apiextensions.k8s.io/antreaagentinfos.clusterinformation.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/antreacontrollerinfos.clusterinformation.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/clusternetworkpolicies.security.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/externalentities.core.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.security.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/tiers.security.antrea.tanzu.vmware.com created
customresourcedefinition.apiextensions.k8s.io/traceflows.ops.antrea.tanzu.vmware.com created
serviceaccount/antctl created
serviceaccount/antrea-agent created
serviceaccount/antrea-controller created
clusterrole.rbac.authorization.k8s.io/aggregate-antrea-policies-edit created
clusterrole.rbac.authorization.k8s.io/aggregate-antrea-policies-view created
clusterrole.rbac.authorization.k8s.io/aggregate-traceflows-edit created
clusterrole.rbac.authorization.k8s.io/aggregate-traceflows-view created
clusterrole.rbac.authorization.k8s.io/antctl created
clusterrole.rbac.authorization.k8s.io/antrea-agent created
clusterrole.rbac.authorization.k8s.io/antrea-controller created
clusterrolebinding.rbac.authorization.k8s.io/antctl created
clusterrolebinding.rbac.authorization.k8s.io/antrea-agent created
clusterrolebinding.rbac.authorization.k8s.io/antrea-controller created
configmap/antrea-ca created
configmap/antrea-config-4h2tb7btgk created
service/antrea created
deployment.apps/antrea-controller created
apiservice.apiregistration.k8s.io/v1alpha1.stats.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta1.controlplane.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta1.networking.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta1.system.antrea.tanzu.vmware.com created
apiservice.apiregistration.k8s.io/v1beta2.controlplane.antrea.tanzu.vmware.com created
daemonset.apps/antrea-agent created
mutatingwebhookconfiguration.admissionregistration.k8s.io/crdmutator.antrea.tanzu.vmware.com created
validatingwebhookconfiguration.admissionregistration.k8s.io/crdvalidator.antrea.tanzu.vmware.com created

```
check the agent and Ccontroller from `Antrea`

`kubectl get pods -n kube-system | grep antrea`

```shell
antrea-agent-b5kpf                                 2/2     Running   0          3m16s
antrea-agent-zfkh9                                 2/2     Running   0          3m16s
antrea-controller-94889cf46-v6rnm                  1/1     Running   0          3m16s
etcd-antreakind-control-plane                      1/1     Running   0          9m42s
kube-apiserver-antreakind-control-plane            1/1     Running   0          9m42s
kube-controller-manager-antreakind-control-plane   1/1     Running   0          9m42s
kube-scheduler-antreakind-control-plane            1/1     Running   0          9m42s
```
also check that coredns pods will change from `Pending` to `Running` state due to plumbing to communications between them

#### go some deep on Antrea

***i):from the output of this command :

`kubectl get all -n kube-system` 

***ii):select one of the `antrea-agent-xxxxx` pod 

***iii):and run this command:

`kubectl exec -n kube-system -it pod/antrea-agent-b5kpf -- ls /`

output sample:

```shell
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-b5kpf -n kube-system' to see all of the containers in this pod.
bin   dev  home  lib	lib64	media  opt   root  sbin  sys  usr
boot  etc  host  lib32	libx32	mnt    proc  run   srv	 tmp  var

```
***iv):enter in shell of agent to play with `OVS` with this command:

`kubectl exec -n kube-system -it pod/antrea-agent-b5kpf -- /bin/bash`

sample output:

```shell
Defaulting container name to antrea-agent.
Use 'kubectl describe pod/antrea-agent-b5kpf -n kube-system' to see all of the containers in this pod.

root@antreakind-control-plane:/# antrea-agent --help
The Antrea agent runs on each node.

Usage:
  antrea-agent [flags]

Flags:
      --add_dir_header                   If true, adds the file directory to the header
      --alsologtostderr                  log to standard error as well as files
      --config string                    The path to the configuration file
  -h, --help                             help for antrea-agent
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --log_file string                  If non-empty, use this log file
      --log_file_max_num uint16          Maximum number of log files per severity level to be kept. Value 0 means unlimited.
      --log_file_max_size uint           Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --logtostderr                      log to standard error instead of files (default true)
      --skip_headers                     If true, avoid header prefixes in the log messages
      --skip_log_headers                 If true, avoid headers when opening log files
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          number for the log level verbosity
      --version                          version for antrea-agent
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
,,,,,,,,,,
,,,,,,,,,,
,,,,,,,,,, 
```

***v): run this command `ovs-<TAB>` to check all options of ovs commands

output example
```shell
ovs-appctl           ovs-docker           ovs-dpctl-top        ovs-parse-backtrace  ovs-pki              ovs-tcpundump        ovs-vsctl            
ovs-bugtool          ovs-dpctl            ovs-ofctl            ovs-pcap             ovs-tcpdump          ovs-vlan-test        ovs-vswitchd  

```
***vi): show `Bridge`

`ovs-vsctl show`
or from terminal we can directly exceute teh command in the container with this command:

`kubectl exec -n kube-system -it pod/antrea-agent-b5kpf -c antrea-agent ovs-vsctl show`

output example:

```shell
d01fff9d-faf3-439f-bb1e-42fe3cfb40a4
    Bridge br-int
        datapath_type: system
        Port antrea-gw0
            Interface antrea-gw0
                type: internal
        Port antrea-tun0
            Interface antrea-tun0
                type: geneve
                options: {csum="true", key=flow, remote_ip=flow}
        Port coredns--975ed0
            Interface coredns--975ed0
    Bridge br-phy
        fail_mode: standalone
        datapath_type: netdev
        Port eth0
            Interface eth0
        Port br-phy
            Interface br-phy
                type: internal
    ovs_version: "2.14.0"
```

***vii): list all ports on OVS `Bridge`

`ovs-vsctl list-port br-int`

output example:

```shell
antrea-gw0
antrea-tun0
coredns--975ed0
root@antreakind-control-plane:/# ovs-vsctl list-br
br-int
br-phy
root@antreakind-control-plane:/# ovs-vsctl list-ports br-int
antrea-gw0
antrea-tun0
coredns--975ed0
root@antreakind-control-plane:/# ovs-vsctl list-ifaces br-int
antrea-gw0
antrea-tun0
coredns--975ed0
```
------------------------------
##### short list of what I used during my exploration and tests
-----------------------------
```shell
ovs-ofctl show br-int
ovs-ofctl dump-ports br-int
ovs-vsctl list-br
ovs-vsctl list-ports br-int
ovs-vsctl list-ifaces br-int
ovs-ofctl dump-flows br-int
ovs-ofctl snoop br-int
ovs-vsctl add-br br-int - set bridge br-int datapath_type=geneve
ovs-vsctl del-br br-int
ovs-vsctl del-controller br-int
ovs-vsctl set Bridge br-int stp_enable=true
ovs-vsctl add-port br-iny ge-1/1/1 type=pronto options:link_speed=1G
ovs-vsctl del-port br-int ge-1/1/
ovs-ofctl add-flow br-int in_port=1,actions=output:2
ovs-ofctl mod-flows br-int in_port=1,dl_type=0x0800,nw_src=100.10.0.1,actions=output:2
ovs-ofctl add-flow br-int in_port=1,actions=output:2,3,4
ovs-ofctl add-flow br-int in_port=1,actions=output:4
ovs-ofctl del-flows br-int
ovs-ofctl mod-port br-int 1 no-flood
ovs-ofctl add-flow br-int in_port=1,dl_type=0x0800,nw_src=192.168.1.241,actions=output:3
ovs-ofctl add-flow br-int in_port=4,dl_type=0x0800,dl_src=60:eb:69:d2:9c:dd,nw_src=198.168.0.2,nw_dst=192.168.0.55,actions=output:1
ovs-ofctl mod-flows br-int in_port=4,dl_type=0x0800,nw_src=192.168.0.45,actions=output:3
ovs-ofctl del-flows br-int in_port=1
```
-------------------------------------------

***viii): in addition to list we can explore the binding with Pod networking with `OVS Bridge`

`ip -c a`

example output
```shell
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ovs-netdev: <BROADCAST,MULTICAST,PROMISC> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 66:20:56:e6:18:21 brd ff:ff:ff:ff:ff:ff
3: br-phy: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 scope global br-phy
       valid_lft forever preferred_lft forever
5: coredns--975ed0@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master ovs-system state UP group default 
    link/ether 56:af:b7:bc:a1:b6 brd ff:ff:ff:ff:ff:ff link-netnsid 1
6: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether c2:d0:42:47:9a:9b brd ff:ff:ff:ff:ff:ff
7: genev_sys_6081: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65000 qdisc noqueue master ovs-system state UNKNOWN group default qlen 1000
    link/ether 7e:6f:3c:4c:84:6f brd ff:ff:ff:ff:ff:ff
8: antrea-gw0: <BROADCAST,MULTICAST> mtu 1450 qdisc noop state DOWN group default qlen 1000
    link/ether ca:83:4c:9f:48:01 brd ff:ff:ff:ff:ff:ff
9: eth0@if10: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

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

Create namespace:

`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml`

output example:
```
namespace/metallb-system created
```
deploy from manifest:

`kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml`

output example
```shell
podsecuritypolicy.policy/controller created
podsecuritypolicy.policy/speaker created
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
role.rbac.authorization.k8s.io/config-watcher created
role.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/config-watcher created
rolebinding.rbac.authorization.k8s.io/pod-lister created
daemonset.apps/speaker created
deployment.apps/controller created
```
check pod creation with this command:

`kubectl -n metallb-system get pods`

output example:

```shell

NAME                              READY   STATUS                       RESTARTS   AGE
pod/controller-65db86ddc6-5zqrf   1/1     Running                      0          57s
pod/speaker-8cjtq                 0/1     CreateContainerConfigError   0          57s
pod/speaker-9gmzm                 0/1     CreateContainerConfigError   0          57s

```

create secret for MetalLB:

`kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"`

then check again statu of Metal LB pods

example output

```shell
kubectl -n metallb-system get pods

NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-65db86ddc6-5zqrf   1/1     Running   0          2m24s
pod/speaker-8cjtq                 1/1     Running   0          2m24s
pod/speaker-9gmzm                 1/1     Running   0          2m24s
```

**Step 7:** Find out the IP addresses being used by the nodes in the K8S cluster:

`kubectl get nodes -o wide`

You'll get output similar to the following:

```shell
NAME                 STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                     KERNEL-VERSION      CONTAINER-RUNTIME
ernesto-control-plane   Ready    master   5m23s   v1.19.1   172.18.0.4    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0
ernesto-worker          Ready    <none>   4m42s   v1.19.1   172.18.0.3    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0
ernesto-worker2         Ready    <none>   4m41s   v1.19.1   172.18.0.2    <none>        Ubuntu Groovy Gorilla (development branch)   4.4.0-185-generic   containerd://1.4.0

```
**Step 8:** Find the [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) of the bridge into tthe cluster

`ip a s`

You'll get output similar to the following

```shell
.
.
.
4: br-956cdadd3d33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:8b:33:c3:fb brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.19.255.255 scope global br-956cdadd3d33
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
using `sipcalc` we can have the range from CIDR to use for the LB depoending on your netwokr settings your will enter differten netwokr segmetn in this example I use `172.18.0.1/16`:

`sipcalc 172.18.0.1/16`

output example:

```shell
-[ipv4 : 172.18.0.1/16] - 0

[CIDR]
Host address		- 172.18.0.1
Host address (decimal)	- 2886860801
Host address (hex)	- AC120001
Network address		- 172.18.0.0
Network mask		- 255.255.0.0
Network mask (bits)	- 16
Network mask (hex)	- FFFF0000
Broadcast address	- 172.18.255.255
Cisco wildcard		- 0.0.255.255
Addresses in network	- 65536
Network range		- 172.18.0.0 - 172.18.255.255
Usable range		- 172.18.0.1 - 172.18.255.254
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
      - 172.18.255.1-172.18.255.250

```

**WHERE**

`172.18.255.1-172.18.255.250` are the IP addreses determined from the steps above.

**Step 13:** Apply the ConfigMap to the K8S cluster

`kubectl apply -f metallb-config.yaml`

**Step 14:** Create a test K8S deployment and bind it to a service

`kubectl create deploy nginx --image nginx`

output example:

```shell
NAME                         READY   STATUS              RESTARTS   AGE
pod/nginx-6799fc88d8-zsmdt   0/1     ContainerCreating   0          2s
```
then expose teh test app:

`kubectl expose deploy nginx --port 80 --type LoadBalancer`
```
service/nginx exposed
```
chekc if this is ok:

`kubectl get all`

output example:

```shell
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-zsmdt   1/1     Running   0          91s

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1      <none>         443/TCP        7h45m
service/nginx        LoadBalancer   10.96.75.187   172.18.255.1   80:31725/TCP   4s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           91s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         1       91s

```

**Step 15:** Confirm that the `nginx` service is bound to an external IP address

`kubectl get services`

You'll get out put similar to the following:

```shell
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
service/kubernetes   ClusterIP      10.96.0.1      <none>         443/TCP        7h45m
service/nginx        LoadBalancer   10.96.75.187   172.18.255.1   80:31725/TCP   4s
```
Notice that the nginx service publishes an `EXTERNAL-IP`.

**Step 16:** Run the `curl` against the `EXTERNAL-IP`, for example:

`curl http://172.18.255.1`

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
Cleanup
`kubectl delete deploy nginx` 
`kubectl delete service nginx`

**Step 16:** Install [Helm](https://helm.sh/)

**Step 17:** Install [Kubeapps]

deploy wordpress from kubeapps using helm

**Step 18:** Install [ARgoCD] and [Concourse]

test with nginx from repo

**Step 19:** Install robot app

**Step 21:** install [TMC]

test netwokr poly deny all and check the trace from antrea plugin on octant

**Step 22:** Install [TO]

do a tracing?

**Step 23:** Install [TSM]

create a global NS and do a GLB between two kind clusters

